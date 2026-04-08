pipeline {
    agent any

    environment {
        // Docker image production
        PROD_API_IMAGE = "hiimwin/oidc-api:prod"
        PROD_CLIENT_IMAGE = "hiimwin/oidc-client:prod"
        // Docker image dev
        DEV_API_IMAGE = "hiimwin/oidc-api:dev"
        DEV_CLIENT_IMAGE = "hiimwin/oidc-client:dev"
        // Compose project
        COMPOSE_PROJECT_NAME = "oidc-app"
    }

    stages {

        stage('Check Docker') {
            steps {
                script {
                    sh 'docker --version'
                    sh 'docker-compose --version'
                }
            }
        }

        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build & Start Docker Compose') {
            steps {
                dir("${env.WORKSPACE}/MyOidcServerDemo") {
                    script {
                        echo "\u001B[34mStopping old containers...\u001B[0m"
                        sh 'docker-compose down -v || true'
                        echo "\u001B[34mBuilding containers...\u001B[0m"
                        sh 'docker-compose build'
                        echo "\u001B[34mStarting containers...\u001B[0m"
                        sh 'docker-compose up -d'
                    }
                }
            }
        }

        stage('Wait for API') {
            steps {
                script {
                    echo "\u001B[33mWaiting for API on localhost:5000...\u001B[0m"
                    def success = false
                    for (int i = 1; i <= 12; i++) {
                        def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:5000", returnStdout: true).trim()
                        if (status == "200") {
                            echo "\u001B[32mAPI is up!\u001B[0m"
                            success = true
                            break
                        } else {
                            echo "\u001B[33mWaiting 5s...\u001B[0m"
                            sleep 5
                        }
                    }
                    if (!success) {
                        error("API did not start in time!")
                    }
                }
            }
        }

        stage('Test Client') {
            steps {
                script {
                    echo "\u001B[33mTesting Client on localhost:5001...\u001B[0m"
                    sh 'curl -s http://localhost:5001 || exit 1'
                }
            }
        }

        stage('Push Docker Images') {
            when {
                expression { return env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'dev' }
            }
            steps {
                dir("${env.WORKSPACE}/MyOidcServerDemo") {
                    script {
                        def apiImage = (env.BRANCH_NAME == 'master') ? PROD_API_IMAGE : DEV_API_IMAGE
                        def clientImage = (env.BRANCH_NAME == 'master') ? PROD_CLIENT_IMAGE : DEV_CLIENT_IMAGE

                        echo "\u001B[34mPushing Docker images for branch: ${env.BRANCH_NAME}\u001B[0m"
                        script {
                            docker.withRegistry('https://docker.io', 'dockerhub-creds') {
                                sh """
                                    docker-compose build
                                    docker tag oidc-app_api:latest ${apiImage}
                                    docker tag oidc-app_client:latest ${clientImage}
                                    docker push ${apiImage}
                                    docker push ${clientImage}
                                """
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "\u001B[32mCI/CD SUCCESS - App is running\u001B[0m"
        }
        failure {
            echo "\u001B[31mCI/CD FAILED - Showing logs...\u001B[0m"
            dir("${env.WORKSPACE}/MyOidcServerDemo") {
                sh 'docker-compose logs || true'
            }
        }
        always {
            echo "\u001B[34mCleaning up containers...\u001B[0m"
            dir("${env.WORKSPACE}/MyOidcServerDemo") {
                sh 'docker-compose down -v || true'
            }
        }
    }
}