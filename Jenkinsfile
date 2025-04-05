pipeline {
    agent any

    environment {
        ARM_CLIENT_ID       = credentials('AZURE_CREDENTIALS_CLIENT_ID')
        ARM_CLIENT_SECRET   = credentials('AZURE_CREDENTIALS_CLIENT_SECRET')
        ARM_SUBSCRIPTION_ID = credentials('AZURE_CREDENTIALS_SUBSCRIPTION_ID')
        ARM_TENANT_ID       = credentials('AZURE_CREDENTIALS_TENANT_ID')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/atulb01/ReactAzureApp.git', branch: 'main'
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
                    bat 'terraform plan -out=tfplan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('infra') {
                    bat 'terraform apply -auto-approve tfplan'
                }
            }
        }

        stage('Build React App') {
            steps {
                dir('react-frontend') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Deploy React App') {
            steps {
                dir('react-frontend') {
                    bat 'az webapp deploy --resource-group YOUR_RESOURCE_GROUP --name YOUR_WEBAPP_NAME --src-path build --type static'
                }
            }
        }
    }
}
