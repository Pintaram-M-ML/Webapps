pipeline {
    agent any

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
                echo "🚀 Running Docker container locally..."
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p 8081:80 ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy using Ansible') {
            steps {
                echo "⚙️ Deploying container to remote server via Ansible..."
                sh 'ansible-playbook -i inventory.ini deploy.yml'
            }
        }

        stage('Post Deployment') {
            steps {
                echo "✅ Application deployed successfully using Jenkins + Ansible!"
            }
        }
    }
}
