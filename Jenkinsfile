pipeline {
    agent any

    environment {
        BRANCH_NAME_SAFE = "${env.BRANCH_NAME.replaceAll('/', '_')}"
        BRANCH_SUFFIX = "${BRANCH_NAME_SAFE}"
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

        stage('Prepare docker-compose for branch') {
            steps {
                script {
                    // Tạo file docker-compose riêng cho branch
                    sh "sed 's/BRANCH_SUFFIX/${BRANCH_SUFFIX}/g' docker-compose.yml > docker-compose.branch.yml"
                    env.COMPOSE_FILE = "${env.WORKSPACE}/docker-compose.branch.yml"
                }
            }
        }

        stage('Parse docker-compose.yml') {
            steps {
                script {
                    def images = sh(script: "docker-compose -f ${env.COMPOSE_FILE} config | grep image: | awk '{print \$2}'", returnStdout: true).trim().split("\n")
                    echo "Detected images: ${images}"
                    env.IMAGES = images.join(' ')
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                dir("${env.WORKSPACE}") {
                    script {
                        // Build client
                        sh "docker build -t oidc-client-${BRANCH_SUFFIX} -f Dockerfile.client ."
                        // Build server
                        sh "docker build -t oidc-server-${BRANCH_SUFFIX} -f Dockerfile.server ."
                    }
                }
            }
        }

        stage('Create Network') {
            steps {
                script {
                    sh "docker network create branch_net-${BRANCH_SUFFIX} || true"
                    echo "Network created: branch_net-${BRANCH_SUFFIX}"
                }
            }
        }

        stage('Start Containers for Test') {
            steps {
                script {
                    sh "docker-compose -f ${env.COMPOSE_FILE} up -d"
                }
            }
        }

        stage('Smoke Test Containers') {
            steps {
                script {
                    // ví dụ test server chạy OK
                    sh "docker ps"
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up containers, network, and dangling images..."
                sh "docker-compose -f ${env.COMPOSE_FILE} down -v || true"
                sh "docker network rm branch_net-${BRANCH_SUFFIX} || true"
                sh "docker system prune -f || true"
            }
        }
    }
}