pipeline {
    agent any
    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_USERNAME = "tristan499"           // ← Changed
        IMAGE_NAME         = "tristan499/jenkinstest"  // ← Updated to your Docker Hub
        IMAGE_TAG          = "latest"
        CONTAINER_NAME     = "jenkinstest"
        HOST_PORT          = "8081"
        CONTAINER_PORT     = "80"
    }

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/tristan499/tristan-repo.git', 
                    branch: 'main'
            }
        }

        stage('Verify Project Files') {
            steps {
                sh '''
                    set -e
                    echo "Checking required project files..."
                    
                    test -f Dockerfile || { echo "Dockerfile not found"; exit 1; }
                    test -f nginx.conf || { echo "nginx.conf not found"; exit 1; }
                    test -f index.html || { echo "index.html not found"; exit 1; }
                    test -f sgustyle.css || { echo "sgustyle.css not found"; exit 1; }
                    test -f sguscript.js || { echo "sguscript.js not found"; exit 1; }
                    test -f grenada-updated.jpeg || { echo "grenada-updated.jpeg not found"; exit 1; }
                    
                    echo "Required files found."
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e
                    docker build --pull -t "$IMAGE_NAME:$IMAGE_TAG" .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        set -e
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    set -e
                    docker push "$IMAGE_NAME:$IMAGE_TAG"
                '''
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    set +e
                    docker rm -f "$CONTAINER_NAME" || true
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    set -e
                    docker run -d \
                      --name "$CONTAINER_NAME" \
                      --restart unless-stopped \
                      -p "$HOST_PORT:$CONTAINER_PORT" \
                      "$IMAGE_NAME:$IMAGE_TAG"
                '''
            }
        }

        stage('Test Website Locally') {
            steps {
                sh '''
                    set -e
                    sleep 3
                    curl -I http://localhost:$HOST_PORT || echo "Warning: Could not reach localhost"
                '''
            }
        }

        stage('Show Running Container') {
            steps {
                sh 'docker ps | grep -E "CONTAINER|$CONTAINER_NAME"'
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
            echo '🌐 Open your EC2 public IP followed by :8081 in a browser.'
        }
        failure {
            echo '❌ Deployment failed. Check the Jenkins console output.'
        }
    }
}
