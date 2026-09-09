pipeline {

    agent any

    tools {
        nodejs 'NodeJS installations'
    }

    environment {
        DOCKER_IMAGE = "mrunmaya22/swiggy"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mrunmayasen/swiggy-nodejs-devops-project.git'
            }
        }

        stage('Node Version') {
            steps {
                sh '''
                    echo "===== Node.js Version ====="
                    node --version

                    echo "===== npm Version ====="
                    npm --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Push') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {

                sh '''
                    echo "===== Applying Kubernetes Deployment ====="

                    kubectl apply -f Kubernetes/deployment.yml

                    echo "===== Applying Kubernetes Service ====="

                    kubectl apply -f Kubernetes/service.yml

                    echo "===== Updating Docker Image ====="

                    kubectl set image deployment/swiggy-app \
                    swiggy-app=${DOCKER_IMAGE}:${BUILD_NUMBER}

                    echo "===== Waiting for Rollout ====="

                    kubectl rollout status deployment/swiggy-app
                '''
            }
        }

        stage('Verify Deployment') {
            steps {

                sh '''
                    echo "===== Deployment ====="
                    kubectl get deployment

                    echo "===== Pods ====="
                    kubectl get pods -o wide

                    echo "===== Service ====="
                    kubectl get service
                '''
            }
        }
    }

    post {

        success {
            echo '''
            ============================================
               SWIGGY APPLICATION DEPLOYED SUCCESSFULLY
            ============================================
            '''
        }

        failure {
            echo '''
            ============================================
               PIPELINE FAILED
               Check the Jenkins Console Output
            ============================================
            '''
        }
    }
}
