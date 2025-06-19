pipeline {
    // agent {
    //     docker {
    //         image 'docker:24.0-cli'
    //         args '--privileged -v /var/run/docker.sock:/var/run/docker.sock -e DOCKER_TLS_CERTDIR=""'
    //     }
    // }

    agent { 
        node {
            label 'docker-agent-builer'
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
                    // Explicitly set Docker host to use mounted socket
                    sh "docker --host unix:///var/run/docker.sock build -t ${IMAGE_NAME}:${BUILD_ID} ."
                }
            }
        }
        
        stage('Run Container') {
            steps {
                script {
                    // Run container using the mounted socket
                    sh """
                        docker --host unix:///var/run/docker.sock run \
                            --name ${CONTAINER_NAME} \
                            --detach \
                            --publish 0:8000 \
                            ${IMAGE_NAME}:${BUILD_ID}
                    """
                    
                    // Get assigned host port
                    PORT = sh(
                        script: "docker --host unix:///var/run/docker.sock inspect --format='{{(index (index .NetworkSettings.Ports \"8000/tcp\") 0).HostPort}}' ${CONTAINER_NAME}",
                        returnStdout: true
                    ).trim()
                    
                    // Wait for container to be healthy (max 2 minutes)
                    timeout(time: 2, unit: 'MINUTES') {
                        waitUntil {
                            def health = sh(
                                script: "docker --host unix:///var/run/docker.sock inspect --format='{{.State.Health.Status}}' ${CONTAINER_NAME}",
                                returnStdout: true
                            ).trim()
                            return health == "healthy"
                        }
                    }
                }
            }
        }
        
        stage('Test Application') {
            steps {
                script {
                    // Test root endpoint
                    sh "curl -f http://localhost:${PORT}/"
                    
                    // Test items endpoint
                    sh "curl -f http://localhost:${PORT}/items/42"
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                script {
                    // Stop and remove container
                    sh "docker --host unix:///var/run/docker.sock stop ${CONTAINER_NAME} || true"
                    sh "docker --host unix:///var/run/docker.sock rm ${CONTAINER_NAME} || true"
                    
                    // Remove image
                    sh "docker --host unix:///var/run/docker.sock rmi ${IMAGE_NAME}:${BUILD_ID} || true"
                }
            }
        }
    }
}