stage('Build React App') {
    steps {
        dir("${REACT_APP_DIR}") {
            bat 'npm install'
            bat 'npm run build'
        }

        // Check if build folder exists
        bat "dir ${REACT_APP_DIR}\\build"

        // Compress build folder contents
        bat """
            powershell -Command "Compress-Archive -Path '${env.WORKSPACE}\\${REACT_APP_DIR}\\build\\*' -DestinationPath '${env.WORKSPACE}\\${BUILD_ZIP}' -Force"
        """

        // Confirm zip file was created
        bat "dir ${env.BUILD_ZIP}"
    }
}

stage('Deploy React App') {
    steps {
        withCredentials([azureServicePrincipal(
            credentialsId: 'azure-sp-jenkins',
            subscriptionIdVariable: 'AZ_SUBSCRIPTION_ID',
            clientIdVariable: 'AZ_CLIENT_ID',
            clientSecretVariable: 'AZ_CLIENT_SECRET',
            tenantIdVariable: 'AZ_TENANT_ID'
        )]) {
            bat """
                az login --service-principal -u %AZ_CLIENT_ID% -p %AZ_CLIENT_SECRET% --tenant %AZ_TENANT_ID%
                az webapp deployment source config-zip ^
                    --resource-group reactjs-rg ^
                    --name reactjs-app-service ^
                    --src ${env.BUILD_ZIP}
            """
        }
    }
}
