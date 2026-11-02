pipeline {
    agent any

    environment {
        IMAGE = 'ramyapadala/my-apache-app'
        CONTAINER = 'my-apache-app'
        SERVER = '192.168.200.170'
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
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'ramyapadala',
                        passwordVariable: 'dckr_pat_1bHLkvXONy1E-uWIscvvZWhao5M'
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
            echo 'Website: http://YOUR_SERVER_IP:8090'
        }

        failure {
            echo 'Deployment failed. Check Jenkins Console Output.'
        }
    }
}

