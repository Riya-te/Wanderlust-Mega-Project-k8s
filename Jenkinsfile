pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = "riuu1110"
        FRONTEND_IMAGE = "riuu1110/wanderlust-frontend"
        BACKEND_IMAGE = "riuu1110/wanderlust-backend"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Riya-te/Wanderlust-Mega-Project-k8s.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=wanderlust \
                    -Dsonar.projectName=Wanderlust \
                    -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh """
                    docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh """
                    docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Trivy Scan Backend') {
            steps {
                sh """
                trivy image --severity HIGH,CRITICAL \
                ${BACKEND_IMAGE}:${BUILD_NUMBER}
                """
            }
        }

        stage('Trivy Scan Frontend') {
            steps {
                sh """
                trivy image --severity HIGH,CRITICAL \
                ${FRONTEND_IMAGE}:${BUILD_NUMBER}
                """
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                    docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}
                    docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                sed -i 's|image: .*wanderlust-backend.*|image: ${BACKEND_IMAGE}:${BUILD_NUMBER}|g' kubernetes/backend.yaml

                sed -i 's|image: .*wanderlust-frontend.*|image: ${FRONTEND_IMAGE}:${BUILD_NUMBER}|g' kubernetes/frontend.yaml
                """
            }
        }

        stage('Push Manifest Changes') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'github-token',
                        variable: 'GITHUB_TOKEN'
                    )
                ]) {
                    sh """
                    git config --global user.email "riyakumari1011.2006@gmail.com"
                    git config --global user.name "Riya Raj"

                    git add .
                    git commit -m "Update image tags ${BUILD_NUMBER}" || true

                    git push https://\$GITHUB_TOKEN@github.com/Riya-te/Wanderlust-Mega-Project-k8s.git HEAD:main
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}