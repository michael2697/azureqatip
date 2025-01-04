pipeline {
    agent any
    environment {
        ARM_CLIENT_ID       = credentials('azure_sp_client_id')
        ARM_CLIENT_SECRET   = credentials('azure_sp_client_secret')
        ARM_SUBSCRIPTION_ID = credentials('azure_subscription_id')
        ARM_TENANT_ID       = credentials('azure_tenant_id')
        TERRAFORM_VERSION   = "1.5.6"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/michael2697/azureqatip.git'
            }
        }
        stage('Install Terraform') {
            steps {
                node {
                    sh '''
                    if ! command -v terraform &> /dev/null; then
                        echo "Terraform not found. Installing..."
                        curl -fsSL https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip -o terraform.zip
                        unzip -o terraform.zip
                        sudo mv terraform /usr/local/bin/
                    else
                        echo "Terraform already installed."
                    fi
                    terraform --version
                    '''
                }
            }
        }
        stage('Azure Login') {
            steps {
                node {
                    sh '''
                    echo "Logging in to Azure CLI..."
                    az login --service-principal \
                        --username $ARM_CLIENT_ID \
                        --password $ARM_CLIENT_SECRET \
                        --tenant $ARM_TENANT_ID
                    az account set --subscription $ARM_SUBSCRIPTION_ID
                    '''
                }
            }
        }
        stage('Terraform Init') {
            steps {
                node {
                    sh '''
                    echo "Initializing Terraform..."
                    terraform init -reconfigure
                    '''
                }
            }
        }
        stage('Validate Terraform') {
            steps {
                node {
                    sh '''
                    echo "Validating Terraform..."
                    terraform validate
                    '''
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                node {
                    sh '''
                    echo "Generating Terraform plan..."
                    terraform plan -out=tfplan
                    '''
                }
            }
        }
        stage('Terraform Apply') {
            steps {
                node {
                    sh '''
                    echo "Applying Terraform configuration..."
                    terraform apply -auto-approve tfplan
                    '''
                }
            }
        }
    }
    post {
        always {
            node {
                echo "Archiving logs and cleaning workspace..."
                archiveArtifacts artifacts: '**/*.log', allowEmptyArchive: true
                archiveArtifacts artifacts: 'tfplan', allowEmptyArchive: true
                cleanWs()
            }
        }
        success {
            echo 'Resources created successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
