
pipeline {
    agent any

    environment {
        HARBOR_CREDENTIALS = 'harbor-creds'
        IMAGE_NAME= '10.212.133.28/uat-jute/cdac_esign:latest'
        GITHUB_CREDENTIALS = 'github-creds-pr'
        DOCKERFILE_PATH = 'cdac-esign-repo'
    }

    stages {
        stage('Clone cdac_esign App') {
            steps {
                dir('cdac-esign-repo') {
                    git branch: 'main', url: 'https://github.com/PratyakashRohila/cdac-esign-repo.git'
                }
            }
        }

        stage('Login to Harbor') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: HARBOR_CREDENTIALS, usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                        sh """
                            echo "$HARBOR_PASS" | docker login 10.212.133.28 -u "$HARBOR_USER" --password-stdin
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dir("${DOCKERFILE_PATH}") {
                        sh """ 
                            docker build \
                            --build-arg http_proxy=http://192.0.2.12:8080 \
                            --build-arg https_proxy=http://192.0.2.12:8080 \
                            --build-arg no_proxy=192.0.2.50:8081 \
                            --build-arg HTTP_PROXY=http://192.0.2.12:8080 \
                            --build-arg HTTPS_PROXY=http://192.0.2.12:8080 \
                            --build-arg NO_PROXY=192.0.2.50:8081 \
                            --tag=${IMAGE_NAME} \
                            --file=Dockerfile \
                            .
                        """
                    }
                }
            }
        }

        stage('Push Docker Image to Harbor') {
            steps{
                sh "docker push ${IMAGE_NAME}"
            }
        }
    }

    post {
        success {
            echo "Build Completed, Image pushed to Harbor!"
        }
        failure {
            echo "Build Failed, Please check Logs!"
        }
        always {
            sh "docker rmi ${IMAGE_NAME} || true"
            sh "docker builder prune -f"
            sh "rm -rf cdac-esign-repo || true"
        }
    }
}
