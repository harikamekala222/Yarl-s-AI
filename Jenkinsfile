pipeline {

    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/Yarl-s-AI"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "======================================"
                echo "CHECKOUT YARL'S AI PROJECT"
                echo "======================================"

                checkout scm
            }
        }

        stage('Copy Project') {
            steps {
                echo "======================================"
                echo "COPYING PROJECT TO EC2"
                echo "======================================"

                sh '''
                    set -e

                    mkdir -p "$PROJECT_DIR"

                    # Remove old application files only.
                    # IMPORTANT: .env is preserved.
                    find "$PROJECT_DIR" -mindepth 1 -maxdepth 1 \
                        ! -name ".env" \
                        -exec rm -rf {} +

                    # Copy Jenkins workspace to deployment directory
                    cp -r "$WORKSPACE"/. "$PROJECT_DIR"/

                    echo "Project copied successfully."
                '''
            }
        }

        stage('Check Environment') {
            steps {
                echo "======================================"
                echo "CHECKING ENVIRONMENT"
                echo "======================================"

                sh '''
                    set -e

                    cd "$PROJECT_DIR"

                    if [ ! -f ".env" ]; then
                        echo "ERROR: .env file does not exist!"
                        echo "Create /home/ubuntu/Yarl-s-AI/.env before running Jenkins."
                        exit 1
                    fi

                    if [ ! -f "docker-compose.yml" ]; then
                        echo "ERROR: docker-compose.yml not found!"
                        exit 1
                    fi

                    if [ ! -f "backend/Dockerfile" ]; then
                        echo "ERROR: backend/Dockerfile not found!"
                        exit 1
                    fi

                    if [ ! -f "front-end/Dockerfile" ]; then
                        echo "ERROR: front-end/Dockerfile not found!"
                        exit 1
                    fi

                    echo "All required files found."

                    echo ""
                    echo "Docker version:"
                    docker --version

                    echo ""
                    echo "Docker Compose version:"
                    docker compose version
                '''
            }
        }

        stage('Stop Previous Containers') {
            steps {
                echo "======================================"
                echo "STOPPING PREVIOUS CONTAINERS"
                echo "======================================"

                sh '''
                    cd "$PROJECT_DIR"

                    docker compose down || true
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "======================================"
                echo "BUILDING DOCKER IMAGES"
                echo "======================================"

                sh '''
                    set -e

                    cd "$PROJECT_DIR"

                    docker compose build --no-cache
                '''
            }
        }

        stage('Start Application') {
            steps {
                echo "======================================"
                echo "STARTING YARL'S AI"
                echo "======================================"

                sh '''
                    set -e

                    cd "$PROJECT_DIR"

                    docker compose up -d

                    echo ""
                    echo "Containers started."
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                echo "======================================"
                echo "VERIFYING CONTAINERS"
                echo "======================================"

                sh '''
                    set -e

                    cd "$PROJECT_DIR"

                    sleep 10

                    echo ""
                    echo "Docker Compose Status:"
                    docker compose ps

                    echo ""
                    echo "Running Containers:"
                    docker ps
                '''
            }
        }

        stage('Application Health Check') {
            steps {
                echo "======================================"
                echo "APPLICATION HEALTH CHECK"
                echo "======================================"

                sh '''
                    cd "$PROJECT_DIR"

                    echo ""
                    echo "Testing Backend - Port 8000..."

                    if curl -f --max-time 10 http://localhost:8000/; then
                        echo "Backend is responding."
                    else
                        echo "WARNING: Backend health check failed."
                    fi

                    echo ""
                    echo "Testing Frontend - Port 4200..."

                    if curl -f --max-time 10 http://localhost:4200/; then
                        echo "Frontend is responding."
                    else
                        echo "WARNING: Frontend health check failed."
                    fi
                '''
            }
        }

        stage('Show Logs') {
            steps {
                echo "======================================"
                echo "APPLICATION LOGS"
                echo "======================================"

                sh '''
                    cd "$PROJECT_DIR"

                    docker compose logs --tail=50
                '''
            }
        }
    }

    post {

        success {
            echo """
======================================
YARL'S AI DEPLOYMENT SUCCESSFUL
======================================

Frontend:
http://YOUR_EC2_PUBLIC_IP:4200

Backend:
http://YOUR_EC2_PUBLIC_IP:8000

Deployment directory:
$PROJECT_DIR

======================================
"""
        }

        failure {
            echo """
======================================
YARL'S AI DEPLOYMENT FAILED
======================================

Check the Jenkins console output.

Run these commands on EC2:

cd /home/ubuntu/Yarl-s-AI

docker compose ps

docker compose logs --tail=100

docker compose logs backend

docker compose logs frontend

======================================
"""
        }

        always {
            echo "======================================"
            echo "JENKINS PIPELINE FINISHED"
            echo "======================================"
        }
    }
}
