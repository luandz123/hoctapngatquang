pipeline {
    agent any

    environment {
        // Defines the image name
        IMAGE_NAME = "spaced-repetition-app"
        REGISTRY = "your-docker-registry" // Replace with your registry (e.g., Docker Hub username)
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout is automatic in Multibranch pipelines, but good to be explicit or just let it be.
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                    sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running smoke tests...'
                    // Simple test: Check if the index.html contains the expected title inside the container
                    sh "docker run --rm ${IMAGE_NAME}:${BUILD_NUMBER} grep 'StudySpacer' /usr/share/nginx/html/index.html"
                }
            }
        }

        stage('Push Image') {
            // Only push if the build and test stages succeed
            when {
                expression { return env.REGISTRY != "your-docker-registry" }
            }
            steps {
                script {
                    echo 'Pushing Docker image to registry...'
                    // Requires docker login credentials configured in Jenkins
                    // withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    //    sh "docker login -u $DOCKER_USER -p $DOCKER_PASS"
                    //    sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    //    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    // }
                    echo "Skipping push: Registry not configured in Jenkinsfile."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying application...'
                    // Stop and remove the existing container if it exists
                    sh "docker rm -f ${IMAGE_NAME} || true"
                    
                    // Run the new container locally on port 8081
                    // This assumes the Jenkins agent has access to the Docker socket
                    sh "docker run -d -p 8081:80 --name ${IMAGE_NAME} ${IMAGE_NAME}:${BUILD_NUMBER}"
                    
                    echo "Application deployed at http://localhost:8081"
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace images to save space
            sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
