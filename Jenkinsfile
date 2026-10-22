
pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'srushtichoudhar/student-service'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKERHUB_REPO:$BUILD_NUMBER .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKERHUB_PASSWORD" | docker login \
                          -u "$DOCKERHUB_USERNAME" --password-stdin
                        docker push "$DOCKERHUB_REPO:$BUILD_NUMBER"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yaml -f service.yaml
                    kubectl set image deployment/student-service \
                      student-service=$DOCKERHUB_REPO:$BUILD_NUMBER
                    kubectl rollout status deployment/student-service
                '''
            }
        }
    }
}
