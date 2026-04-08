pipeline {
    agent any
    environment {
        BRANCH_TAG = "${env.BRANCH_NAME.replaceAll('/', '_')}" // fix tên branch cho Docker tag
    }
    stages {
        stage('Checkout') {
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
        stage('Parse docker-compose.yml') {
            steps {
                script {
                    // Lấy tên image từ docker-compose.yml
                    IMAGE_NAMES = sh(
                        script: "docker-compose config | grep image: | awk '{print \$2}'",
                        returnStdout: true
                    ).trim().split('\n')

                    echo "Detected images: ${IMAGE_NAMES}"

                    // Thêm branch suffix
                    IMAGE_NAMES_BRANCH = IMAGE_NAMES.collect { it + '-' + BRANCH_TAG }
                    echo "Images with branch suffix: ${IMAGE_NAMES_BRANCH}"
                }
            }
        }
        stage('Build Docker Images') {
            steps {
                dir("${WORKSPACE}") {
                    script {
                        IMAGE_NAMES.eachWithIndex { image, idx ->
                            sh "docker build -t ${IMAGE_NAMES_BRANCH[idx]} -f Dockerfile.${image.contains('server') ? 'server' : 'client'} ."
                        }
                    }
                }
            }
        }
        stage('Start Containers for Test') {
            steps {
                dir("${WORKSPACE}") {
                    script {
                        // Tạo docker-compose override runtime với tag branch
                        def composeContent = readFile('docker-compose.yml')
                        IMAGE_NAMES.eachWithIndex { image, idx ->
                            composeContent = composeContent.replaceAll(
                                "(image:\\s*${image})", "image: ${IMAGE_NAMES_BRANCH[idx]}"
                            )
                        }
                        writeFile file: 'docker-compose.branch.yml', text: composeContent

                        sh "docker-compose -f docker-compose.branch.yml up -d"
                        sh "sleep 5"
                    }
                }
            }
        }
        stage('Smoke Test Containers') {
            steps {
                script {
                    echo "Running basic smoke tests..."
                    // ví dụ test server endpoint
                    sh "curl -f http://localhost:5000/.well-known/openid-configuration"
                }
            }
        }
    }
    post {
        always {
            dir("${WORKSPACE}") {
                echo "Cleaning up containers and dangling images..."
                sh "docker-compose -f docker-compose.branch.yml down -v"
                sh "docker system prune -f"
            }
        }
    }
}