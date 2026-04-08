pipeline {
    agent any

    environment {
        // Tạo branch-safe tên image/network
        BRANCH_NAME_SAFE = "${env.BRANCH_NAME.replaceAll('/', '_')}"
        IMAGE_SERVER = "oidc-server-${BRANCH_NAME_SAFE}"
        IMAGE_CLIENT = "oidc-client-${BRANCH_NAME_SAFE}"
        COMPOSE_FILE = "docker-compose.branch.yml"
        NETWORK_NAME = "net-${BRANCH_NAME_SAFE}"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Check Docker & Compose') {
            steps {
                sh 'docker --version'
                sh 'docker-compose --version'
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('.') {
                    sh "docker build -t ${IMAGE_CLIENT} -f Dockerfile.client ."
                    sh "docker build -t ${IMAGE_SERVER} -f Dockerfile.server ."
                }
            }
        }

        stage('Start Containers for Test') {
            steps {
                script {
                    sh "docker network create ${NETWORK_NAME} || true"
                    sh "docker-compose -f ${COMPOSE_FILE} up -d"
                    // Chờ container khởi động
                    sh "sleep 5"
                }
            }
        }

        stage('Smoke Test Containers') {
            steps {
                script {
                    echo "Running basic smoke tests..."
                    // Chạy curl từ một container riêng trong cùng network
                    sh """docker run --rm --network ${NETWORK_NAME} curlimages/curl:latest \
                        -f http://oidc-server:80/.well-known/openid-configuration"""
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up containers, network, and dangling images..."
            sh "docker-compose -f ${COMPOSE_FILE} down -v"
            sh "docker network rm ${NETWORK_NAME} || true"
            sh "docker system prune -f"
        }
    }
}