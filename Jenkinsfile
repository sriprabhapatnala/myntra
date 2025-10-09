pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "prabha023/myntra"   // change to your DockerHub repo
        IMAGE_TAG = "v1"
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
                     echo "Building Docker image..."
                     docker build -t $DOCKERHUB_REPO:$IMAGE_TAG .
                     '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo "Logging into DockerHub..."
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push $DOCKERHUB_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

       stage('Run Container') {
    steps {
        sh '''
          echo "Running Myntra container locally..."

          # Stop and remove old container
          docker rm -f myntra-container || true

          # Free up port 8081 (stop swarm service if using it)
          docker service rm myntra || true

          # Run new container
          docker run -d --name myntra-container -p 8081:80 $DOCKERHUB_REPO:$IMAGE_TAG

          echo "Myntra container is up at http://<server-ip>:8081"
        '''
    }
}

        stage('Deploy to Docker Swarm') {
            steps {
                sh '''
                  echo "Deploying Myntra app on Swarm..."

                  # initialize swarm if not already
                  docker swarm init || true

                  # remove old service if exists
                  docker service rm myntra || true

                  # create new service with exposed port 8081
                  docker service create \
                    --name myntra \
                    --publish 8081:80 \
                    $DOCKERHUB_REPO:$IMAGE_TAG

                  echo "Myntra service deployed successfully!"
                '''
            }
        }
    }
}
