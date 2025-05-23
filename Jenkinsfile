def webServerIp = "65.0.132.46"  // IP yaha 

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'dugreshyadav/ci-cd-test-app'
        EC2_HOST = 'ec2-user@15.206.172.142'
        EC2_KEY = credentials('ec2-ssh-key')  // Add in Jenkins > Credentials
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/durgeshyadavwork/ci-cd-test-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh """
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push $DOCKER_IMAGE:${BUILD_NUMBER}
                        
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${webServerIp} '
                        docker pull $DOCKER_IMAGE:${BUILD_NUMBER} &&
                        docker stop app || true &&
                        docker rm app || true &&
                        docker run -d --name app -p 3000:3000 $DOCKER_IMAGE:${BUILD_NUMBER}
                    '
                    """
                }
            }
        }
        stage('Cleanup') {
            steps {
                sh 'docker rmi $DOCKER_IMAGE:${BUILD_NUMBER}'
            }
        }
    }
}
