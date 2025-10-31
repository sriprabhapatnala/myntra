pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'prabha023/myntra'   // Your DockerHub repo
        IMAGE_TAG = "v${BUILD_NUMBER}"        // Auto-tag with Jenkins build number
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sriprabhapatnala/myntra.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  echo "=== Building Docker image ==="
                  docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "=== Logging into DockerHub ==="
                      echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                      
                      echo "=== Pushing Docker image ==="
                      docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                      
                      echo "Docker image pushed: ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                  echo "=== Running Myntra container locally ==="
                  
                  # Stop and remove old container if exists
                  docker rm -f myntra-container || true
                  
                  # Run the new container
                  docker run -d --name myntra-container -p 8081:80 ${DOCKERHUB_REPO}:${IMAGE_TAG}
                  
                  echo "Myntra container is running on http://<server-ip>:8081"
                '''
            }
        }

        stage('Deploy to Docker Swarm') {
            steps {
                sh '''
                  echo "=== Deploying Myntra service to Docker Swarm ==="
                  
                  # Initialize swarm if not already initialized
                  docker swarm init 2>/dev/null || true
                  
                  # Remove old service if it exists
                  docker service rm myntra 2>/dev/null || true
                  
                  # Deploy new service
                  docker service create \
                    --name myntra \
                    --publish 8081:80 \
                    ${DOCKERHUB_REPO}:${IMAGE_TAG}
                  
                  echo "Myntra service deployed successfully!"
                '''
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline failed! Check the logs for errors."
        }
        success {
            echo "✅ Pipeline executed successfully — Myntra app deployed on Swarm!"
        }
    }
}
