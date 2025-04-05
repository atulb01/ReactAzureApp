pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS = credentials('azure-sp-jenkins') // This can be removed if using withCredentials block instead
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/atulb01/ReactAzureApp.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('infra') {
                    bat 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('infra') {
                    bat 'terraform plan -var-file=terraform.tfvars'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('infra') {
                    bat 'terraform apply -auto-approve -var-file=terraform.tfvars'
                }
            }
        }

        stage('Build React App') {
            steps {
                bat 'npm install'
                bat 'npm run build'
                // Create zip from build folder
                bat 'powershell Compress-Archive -Path build\\* -DestinationPath build.zip'
            }
        }

        stage('Deploy React App') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'AZURE_CREDENTIALS_ID',
                    subscriptionIdVariable: 'AZ_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZ_CLIENT_ID',
                    clientSecretVariable: 'AZ_CLIENT_SECRET',
                    tenantIdVariable: 'AZ_TENANT_ID'
                )]) {
                    bat '''
                    az login --service-principal -u %AZ_CLIENT_ID% -p %AZ_CLIENT_SECRET% --tenant %AZ_TENANT_ID%
                    az webapp deployment source config-zip ^
                        --resource-group reactjs-rg ^
                        --name reactjs-app-service ^
                        --src build.zip
                    '''
                }
            }
        }
    }
}
