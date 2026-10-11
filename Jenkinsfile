pipeline {
    agent any

    environment {
        IMAGE_NAME = 'PodaMounitha1/vle7-nginx'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'test -s index.html'
                sh 'grep -q "VLE-7 DevOps Monitoring System" index.html'
                sh 'test -f Dockerfile'
                sh 'test -f k8s/deployment.yaml'
                sh 'test -f k8s/service.yaml'
                echo 'Application checks passed'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" --password-stdin
                        docker push "${IMAGE_NAME}:${BUILD_NUMBER}"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/vle7-nginx \
                      nginx="${IMAGE_NAME}:${BUILD_NUMBER}"

                    kubectl rollout status deployment/vle7-nginx \
                      --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get deployment,pods,service'
            }
        }
    }
}
