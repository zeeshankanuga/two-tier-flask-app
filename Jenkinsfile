pipeline {
    agent { label 'dev' } // Specifies where the entire pipeline runs

    stages {
        stage('code') {
            steps {
                git url: "https://github.com/zeeshankanuga/two-tier-flask-app.git", branch: 'master'
            }
        }
        stage('build') {
            steps {
                // 1. Force a build from scratch ignoring all previous layers
                sh 'docker build --no-cache -t flask-app:latest .'
            }
        }
        stage('Test') {
            steps {
                // Steps for the test stage (e.g., run unit tests, aggregate reports)
                echo "tester likhkar dega"
            }
        }
        stage('Push to Docker Hub') {
            environment {
                DOCKER_HUB_USER = 'zeeshankanuga' // Replace with your Docker Hub username
                IMAGE_NAME = 'flask-app'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    // 1. Log in to Docker Hub
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    
                    // 2. Tag the image (Ensure the tag matches your hub repo: username/repo:tag)
                    sh "docker tag ${IMAGE_NAME}:latest ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    sh "docker tag ${IMAGE_NAME}:latest ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
        
                    // 3. Push the images
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
        stage('Trivy Security Scan') {
            steps {
                script {
                    // Scan the image and output results to the console
                    sh "trivy image --severity HIGH,CRITICAL zeeshankanuga/flask-app:latest"
                }
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                docker compose down
                docker compose pull
                docker compose up -d --force-recreate
                '''
            }
        }
        stage('Cleanup') {
            steps {
                // Removes old images that are no longer associated with a container
                sh 'docker image prune -f'
            }
        }
    }

    // Optional: Actions to run after the pipeline finishes (e.g., cleanup, notifications)
    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
