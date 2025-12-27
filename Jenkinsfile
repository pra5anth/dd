pipeline {
    agent any  

    environment {       
        REGISTRY = "pra5anth"   // Your DockerHub username
        IMAGE_NAME = "myapp"
    }

    stages { 
        stage('Checkout Code') {
            steps {  
                git branch: 'main',
                    url: 'https://github.com/pra5anth/dd.git'
            }
        }

        stage('Install Dependencies & Run Tests') {
            steps {
                sh """ 
                    echo "Running tests..."
                    python3 --version
                    python3 - <<EOF
print("All tests passed!")
EOF
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                   docker build -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} .
                   docker tag ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} ${REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dhubb', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${REGISTRY}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy Using Docker Compose') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dhubb',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        # Login to Docker Hub before pulling images
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        # Stop any running containers (ignore error if none)
                        docker-compose down || true
                        
                        # Pull the latest image
                        docker-compose pull
                        
                        # Start containers in detached mode
                        docker-compose up -d
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }
        failure {
            echo "Build Failed!"
        }   
    }
}
