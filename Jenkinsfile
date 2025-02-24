pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-southeast-1'
        CLUSTER_NAME = 'dev-cluster'
        K8S_MANIFEST_PATH = 'k8s-manifest-files' // Path to the manifest files in the workspace
    }
    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out the repository..."
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], userRemoteConfigs: [[url: 'git@github.com:jaesukdo1986/assessment.git']]])
            }
        }
        stage('Setup AWS CLI and kubectl') {
            steps {
                script {
                    sh '''
                    echo "Installing AWS CLI and kubectl if not already installed..."

                    # Install AWS CLI if not present
                    if ! command -v aws &> /dev/null; then
                        echo "AWS CLI not found, installing..."
                        curl "https://awscli.amazonaws.com/aws-cli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                        unzip awscliv2.zip
                        sudo ./aws/install
                    fi

                    # Install kubectl if not present
                    if ! command -v kubectl &> /dev/null; then
                        echo "kubectl not found, installing..."
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        sudo mv kubectl /usr/local/bin/
                    fi
                    '''
                }
            }
        }
        stage('Configure AWS CLI') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    script {
                        sh '''
                        echo "Configuring AWS CLI..."
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region ${AWS_REGION}
                        '''
                    }
                }
            }
        }
        stage('Authenticate with EKS Cluster') {
            steps {
                script {
                    sh '''
                    echo "Authenticating with EKS Cluster..."
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                    '''
                }
            }
        }
        stage('Deploy Weather App') {
            steps {
                script {
                    sh '''
                    echo "Deploying Weather App..."
                    kubectl apply -f ${K8S_MANIFEST_PATH}
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
