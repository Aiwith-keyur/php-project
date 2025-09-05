pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Jenkins credential ID
        DOCKERHUB_USER = "YOUR_DOCKERHUB_USERNAME"              // replace with your DockerHub username
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Aiwith-keyur/php-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def img = docker.build("${DOCKERHUB_USER}/php-project:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
                        def img = docker.image("${DOCKERHUB_USER}/php-project:${BUILD_NUMBER}")
                        img.push()
                        img.push("latest")  // also tag as latest
                    }
                }
            }
        }

        stage('Deploy on Node') {
            agent { label 'deploy-node' }  // make sure you created a node with this label
            steps {
                sh '''
                    docker pull ${DOCKERHUB_USER}/php-project:${BUILD_NUMBER}
                    docker rm -f phpproject || true
                    docker run -d --name phpproject -p 8083:80 ${DOCKERHUB_USER}/php-project:${BUILD_NUMBER}
                '''
            }
        }
    }
}

