```groovy
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
                sh 'npm test -- --watchAll=false --passWithNoTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "===== Building Docker Image ====="

                    docker build \
                    -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .

                    echo "===== Docker Image Built Successfully ====="

                    docker images | grep ${DOCKER_IMAGE}
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
                        echo "===== Logging into Docker Hub ====="

                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin

                        echo "===== Pushing Docker Image ====="

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                        echo "===== Docker Image Pushed Successfully ====="

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

                    echo "===== Kubernetes Deployment Successful ====="
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

                    echo "===== Current Docker Image ====="
                    kubectl describe deployment swiggy-app | grep -i image
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
```
