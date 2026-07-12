pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/go-web-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_REPO = "https://github.com/your-username/go-web-app.git"
        BRANCH = "main"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build') {
            steps {
                sh 'go build -o go-web-app'
            }
        }

        stage('Test') {
            steps {
                sh 'go test ./...'
            }
        }

        stage('Code Quality') {
            steps {
                sh 'golangci-lint run'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
            }
        }

        stage('Update Helm Chart') {
            steps {
                sh """
                sed -i 's/tag:.*/tag: "${IMAGE_TAG}"/' helm/go-web-app-chart/values.yaml
                """
            }
        }

        stage('Commit & Push Helm Changes') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh """
                    git config --global user.email "your-email@example.com"
                    git config --global user.name "Jenkins"

                    git add helm/go-web-app-chart/values.yaml
                    git commit -m "Update image tag to ${IMAGE_TAG}" || true

                    git push https://${GIT_USER}:${GIT_TOKEN}@github.com/your-username/go-web-app.git HEAD:main
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}