pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS = credentials('AZURE_CREDENTIALS_ID') // Add in Jenkins Credentials
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
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('infra') {
                    sh 'terraform plan -var-file=terraform.tfvars'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('infra') {
                    sh 'terraform apply -auto-approve -var-file=terraform.tfvars'
                }
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm install'
                sh 'npm run build'
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
                    sh '''
                        az login --service-principal -u $AZ_CLIENT_ID -p $AZ_CLIENT_SECRET --tenant $AZ_TENANT_ID
                        az webapp deployment source config-zip \
                            --resource-group reactjs-rg \
                            --name reactjs-app-service \
                            --src build.zip
                    '''
                }
            }
        }
    }
}
