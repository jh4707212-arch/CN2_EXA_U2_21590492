pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "quesito/crud_estudiantes"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials" 
        REMOTE_HOST = "juan@45.55.64.106"
        STACK_NAME = "stack_proyecto"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = sh(returnStdout: true, script: 'git describe --tags --always | tr -d "\n"').trim()
                    env.IMAGE_TAG = tag ? tag : "latest"
                    sh "docker build -t ${IMAGE_NAME}:${env.IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${env.IMAGE_TAG}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Swarm') {
            steps {
                script {
                    // Crea la carpeta en el host y transfiere el compose actualizado
                    sh "ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} 'mkdir -p /home/juan/crud'"
                    sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${REMOTE_HOST}:/home/juan/crud/docker-compose.yaml"
                    // Executa el despliegue desde el shell del Manager
                    sh "ssh -o StrictHostKeyChecking=no ${REMOTE_HOST} 'docker stack deploy -c /home/juan/crud/docker-compose.yaml ${STACK_NAME}'"
                }
            }
        }
    }

    post {
        success {
            echo '¡Proyecto desplegado con éxito en el clúster Swarm!'
        }
        failure {
            echo 'Ocurrió un error durante la ejecución del pipeline.'
        }
    }
}   