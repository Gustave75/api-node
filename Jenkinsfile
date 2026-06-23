pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "gustavedev/api-node:latest"
        AWS_HOST = "13.48.148.134"
        AWS_USER = "ubuntu"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Installation des dependances...'
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }
        stage('Push DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push ${DOCKER_IMAGE}'
                }
            }
        }
        stage('Deploy sur AWS') {
            steps {
                sshagent(['aws-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${AWS_USER}@${AWS_HOST} \
                        "sudo docker pull ${DOCKER_IMAGE} && \
                         sudo docker stop api-node || true && \
                         sudo docker rm api-node || true && \
                         sudo docker run -d --name api-node -p 3000:3000 ${DOCKER_IMAGE}"
                    '''
                }
            }
        }
    }
}
