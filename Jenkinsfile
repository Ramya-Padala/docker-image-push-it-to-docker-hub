
pipeline {
    agent any

    environment {
        IMAGE       = 'ramyapadala/my-apache-app'
        CONTAINER   = 'my-apache-app'
        SERVER      = '192.168.201.140'
        DEPLOY_USER = 'vitel'
        PORT        = '8090'
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
                        set +x

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
                ssh -o BatchMode=yes \
                    -o IdentitiesOnly=yes \
                    vitel@192.168.201.140 "
                        docker pull ramyapadala/my-apache-app:latest &&
                        (docker rm -f my-apache-app || true) &&
                        docker run -d \
                            --name my-apache-app \
                            --restart unless-stopped \
                            -p 8090:80 \
                            ramyapadala/my-apache-app:latest
                    "
            '''
        }
    }
}

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 5
                    curl -f http://$SERVER:$PORT/
                '''
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


