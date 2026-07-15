pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven3'
        nodejs 'NodeJS20'
    }

    environment {
        DOCKER_USERNAME = "varungummadi"

        BACKEND_IMAGE = "${DOCKER_USERNAME}/smartims-backend"
        FRONTEND_IMAGE = "${DOCKER_USERNAME}/smartims-frontend"

        IMAGE_TAG = "${BUILD_NUMBER}"

        // SONAR_HOME = tool 'SonarQube'
    }

    stages {

        stage('Code Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/VarunGummadi/smart-incident-management.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Unit Test') {
            steps {
                dir('backend') {
                    sh 'mvn test'
                }
            }
        }

        //stage('Code Quality Test') {
           // steps {
                //withSonarQubeEnv('SonarQube') {
                   // dir('backend') {
                       // sh '''
                       // mvn sonar:sonar \
                       // -Dsonar.projectKey=smartims \
                       // -Dsonar.projectName=smartims
                       // '''
                    //}
               // }
            //}
        //}

        stage('Build Docker Images') {
            steps {

                dir('backend') {
                    sh "docker build -t ${BACKEND_IMAGE}:${IMAGE_TAG} ."
                }

                dir('frontend') {
                    sh "docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Tag Docker Images') {
            steps {

                sh """
                docker tag ${BACKEND_IMAGE}:${IMAGE_TAG} ${BACKEND_IMAGE}:latest
                docker tag ${FRONTEND_IMAGE}:${IMAGE_TAG} ${FRONTEND_IMAGE}:latest
                """
            }
        }

        stage('Push Docker Images') {

            steps {

                withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push '${BACKEND_IMAGE}:${IMAGE_TAG}'
                    docker push '${BACKEND_IMAGE}:latest'

                    docker push '${FRONTEND_IMAGE}:${IMAGE_TAG}'
                    docker push '${FRONTEND_IMAGE}:latest'

                    docker logout
                    """
                }

            }

        }

        stage('Deploy to Kubernetes') {

            steps {

                sh '''
                kubectl apply -f kubernetes/namespace.yaml
                
                kubectl apply -f kubernetes/

                kubectl rollout restart deployment smartims-backend

                kubectl rollout restart deployment smartims-frontend
                '''
            }

        }

        stage('Verify Deployment') {

            steps {

                sh '''
                kubectl get pods

                kubectl get svc

                kubectl rollout status deployment/smartims-backend

                kubectl rollout status deployment/smartims-frontend
                '''
            }

        }

    }

    post {

        success {

            echo "Pipeline executed successfully."

        }

        failure {

            echo "Pipeline failed."

        }

        always {

            cleanWs()

        }

    }

}