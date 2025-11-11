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
        withCredentials([azureServicePrincipal('jenkins-sp')]) {
            sh '''
                echo "Logging into Azure..."
                az login --service-principal \
                    -u $AZURE_CLIENT_ID \
                    -p $AZURE_CLIENT_SECRET \
                    --tenant $AZURE_TENANT_ID

                az account set --subscription $AZURE_SUBSCRIPTION_ID
                echo "✅ Azure login successful."
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

        stage('Set kubeconfig') {
    steps {
        echo "🔧 Setting KUBECONFIG for Jenkins..."
        sh 'export KUBECONFIG=/var/lib/jenkins/.kube/config && kubectl get nodes'
    }
}

stage('Deploy with Helm') {
    steps {
        echo "🚀 Deploying application with Helm..."
        sh '''
            export KUBECONFIG=/var/lib/jenkins/.kube/config
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
        echo "⚙️ Deploying container via Ansible..."
        sh '''
            export KUBECONFIG=/var/lib/jenkins/.kube/config
            export ANSIBLE_HOST_KEY_CHECKING=False
            ansible-playbook -i inventory.ini deploy.yml
        '''
    }
}



        stage('Post Deployment') {
            steps {
                echo "✅ Application deployed successfully using Jenkins + Ansible!"
            }
        }
    }
}
