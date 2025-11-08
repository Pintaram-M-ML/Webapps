pipeline {
    agent any

    environment {
        IMAGE_NAME = "pintaram369/my-app-image"
        IMAGE_TAG = "${BUILD_NUMBER}"
        HELM_RELEASE = "my-app"
        HELM_CHART_PATH = "./helm-chart"   // path to your Helm chart folder
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
        KUBE_NAMESPACE = "default"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📦 Cloning repository..."
                git branch: 'main', url: 'https://github.com/Pintaram-M-ML/Webapps.git'
            }
        }

       stage('Login to Azure') {
    steps {
        withCredentials([
            string(credentialsId: 'AZ_CLIENT_ID', variable: 'AZ_CLIENT_ID'),
            string(credentialsId: 'AZ_CLIENT_SECRET', variable: 'AZ_CLIENT_SECRET'),
            string(credentialsId: 'AZ_TENANT_ID', variable: 'AZ_TENANT_ID'),
            string(credentialsId: 'AZ_SUBSCRIPTION_ID', variable: 'AZ_SUBSCRIPTION_ID')
        ]) {
            sh '''
                echo "🔑 Logging into Azure..."
                mkdir -p /tmp/azure
                export AZURE_CONFIG_DIR=/tmp/azure

                az login --service-principal \
                    -u $AZ_CLIENT_ID \
                    -p $AZ_CLIENT_SECRET \
                    --tenant $AZ_TENANT_ID \
                    --output none \
                    --only-show-errors

                az account set --subscription $AZ_SUBSCRIPTION_ID
                echo "✅ Azure login successful"
            '''
        }
    }
}




        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "📤 Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                echo "🚀 Deploying application with Helm..."
                sh '''
                    helm upgrade --install ${HELM_RELEASE} ${HELM_CHART_PATH} \
                        --namespace ${KUBE_NAMESPACE} \
                        --set image.repository=${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG} \
                        --wait --timeout 120s
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
