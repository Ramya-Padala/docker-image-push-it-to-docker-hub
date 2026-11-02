pipeline {
    agent any

    environment {
        IMAGE = 'ramyapadala/my-apache-app'
        CONTAINER = 'my-apache-app'
        SERVER = '192.168.201.140'
        PORT = '8090' 
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Ramya-Padala/docker-image-push-it-to-docker-hub.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DH_PASS" | docker login \
                            -u "$DH_USER" --password-stdin

                        docker push $IMAGE:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on Remote Server') {
            steps {
                sshagent(['deploy-server-ssh']) {
                    sh '''
                        ssh -o BatchMode=yes $SERVER "
                            docker pull $IMAGE:latest &&
                            docker rm -f $CONTAINER || true
                        "

                        ssh -o BatchMode=yes $SERVER "
                            docker run -d \
                                --name $CONTAINER \
                                --restart unless-stopped \
                                -p $PORT:80 \
                                $IMAGE:latest
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
            echo 'Website: http://192.168.201.140:8090'
        }

        failure {
            echo 'Deployment failed. Check Jenkins Console Output.'
        }
    }
}

