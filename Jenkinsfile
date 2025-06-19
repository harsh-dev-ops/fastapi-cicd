pipeline {
    agent {
        docker {
            image 'docker:24.0-cli'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u root'
        }
    }
    
    environment {
        IMAGE_NAME = "fastapi-app"
        CONTAINER_NAME = "fastapi-test-${BUILD_ID}"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${BUILD_ID}")
                }
            }
        }
        
        stage('Run Container') {
            steps {
                script {
                    // Run container on dynamic port
                    docker.image("${IMAGE_NAME}:${BUILD_ID}").run(
                        "--name ${CONTAINER_NAME} " +
                        "--detach " +
                        "--publish 8000:8000"
                    )
                    
                    // Get assigned host port
                    PORT = sh(
                        script: "docker inspect --format='{{(index (index .NetworkSettings.Ports \"8000/tcp\") 0).HostPort}}' ${CONTAINER_NAME}",
                        returnStdout: true
                    ).trim()
                    
                    // Wait for container to be healthy
                    timeout(time: 1, unit: 'MINUTES') {
                        waitUntil {
                            sh "docker inspect --format='{{.State.Health.Status}}' ${CONTAINER_NAME} | grep -q 'healthy'"
                        }
                    }
                }
            }
        }
        
        stage('Test Application') {
            steps {
                script {
                    // Test root endpoint
                    sh "curl -s http://localhost:${PORT}/ | grep '\"message\":\"Hello from FastAPI in Docker!\"'"
                    
                    // Test items endpoint
                    sh "curl -s http://localhost:${PORT}/items/42 | grep '\"item_id\":42'"
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                script {
                    // Stop and remove container
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    
                    // Remove image
                    sh "docker rmi ${IMAGE_NAME}:${BUILD_ID} || true"
                }
            }
        }
    }
}