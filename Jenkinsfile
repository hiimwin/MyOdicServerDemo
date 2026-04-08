pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "oidc-app"
        REPO_API = "hiimwin/oidc-api"
        REPO_CLIENT = "hiimwin/oidc-client"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Run with Docker Compose') {
            steps {
                script {
                    // Build & run all containers
                    sh """
                    echo "Stopping old containers..."
                    docker-compose down -v || true

                    echo "Building & starting containers..."
                    docker-compose build
                    docker-compose up -d
                    """
                }
            }
        }

        stage('Wait for API') {
            steps {
                sh '''
                echo "Waiting for API to be ready..."
                for i in {1..12}; do
                    curl -s http://localhost:5000 && break
                    echo "Waiting 5s..."
                    sleep 5
                done
                '''
            }
        }

        stage('Test Client') {
            steps {
                sh '''
                echo "Testing Client..."
                curl -s http://localhost:5001 || exit 1
                '''
            }
        }

        stage('Push Images (Master Only)') {
            when { branch 'master' }
            steps {
                script {
                    docker.withRegistry('https://docker.io', 'dockerhub-creds') {
                        sh '''
                        docker-compose build
                        docker tag oidc-app_api:latest hiimwin/oidc-api:latest
                        docker tag oidc-app_client:latest hiimwin/oidc-client:latest
                        docker push hiimwin/oidc-api:latest
                        docker push hiimwin/oidc-client:latest
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD SUCCESS - App is running"
        }
        failure {
            echo "CI/CD FAILED - Showing logs..."
            sh 'docker-compose logs'
        }
        always {
            echo "Cleaning up containers..."
            sh 'docker-compose down -v || true'
        }
    }
}