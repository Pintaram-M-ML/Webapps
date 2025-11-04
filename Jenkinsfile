pipeline {
    // ✅ Define agent properly with custom workspace
    agent {
        any {
            customWorkspace '/home/jenkins_home/workspace/Integration_webapp'
        }
    }

    environment {
        IMAGE_NAME = "webimage"
        CONTAINER_NAME = "webcontainer"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📦 Cloning repository..."
                git branch: 'main', url: 'https://github.com/Pintaram-M-ML/Webapps.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Run Docker Container') {
            steps {
                echo "🚀 Running Docker container..."
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p 8080:80 ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Post Deployment') {
            steps {
                echo "✅ Application deployed successfully!"
            }
        }
    }
}
