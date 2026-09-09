pipeline {
agent any

```
environment {
    DOCKER_IMAGE = "mrunmaya22/swiggy"
}

stages {

    stage('Checkout') {
        steps {
            git 'https://github.com/mrunmayasen/swiggy-nodejs-devops-project.git'
        }
    }

    stage('Install & Test') {
        steps {
            sh 'npm install'
            sh 'npm test -- --watchAll=false'
        }
    }

    stage('Build Docker Image') {
        steps {
            sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
        }
    }

    stage('Push to Docker Hub') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'msrout22@gmail.com',
                    passwordVariable: 'Bedu.1234'
                )
            ]) {
                sh '''
                    echo "$DOCKER_PASSWORD" | docker login \
                    -u "$DOCKER_USERNAME" \
                    --password-stdin

                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    docker logout
                '''
            }
        }
    }

    stage('Deploy to Kubernetes') {
        steps {
            sh """
                kubectl apply -f Kubernetes/deployment.yml
                kubectl apply -f Kubernetes/service.yml

                kubectl set image deployment/swiggy-app \
                swiggy-app=${DOCKER_IMAGE}:${BUILD_NUMBER}

                kubectl rollout status deployment/swiggy-app
            """
        }
    }
}

post {
    success {
        echo "Swiggy application deployed successfully!"
        echo "Docker Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
    }

    failure {
        echo "Pipeline failed. Check the Jenkins console output."
    }
}
```

}
