pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal-py'
        RESOURCE_GROUP = 'python-webapp-rg'
        APP_SERVICE_NAME = 'python-webapp-service-adi-7'
        PYTHON_VERSION = '3.11'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                 git branch: 'master', url: 'https://github.com/aditya-blanko/WebApiJenkins.git'
            }
        }
        
        stage('Build') {
            steps {
                bat 'python -m pip install --upgrade pip'
                bat 'pip install -r requirements.txt'
                bat 'pip install pytest'
                bat 'python -m pytest'
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    bat "az group create --name $RESOURCE_GROUP --location eastus"
                    bat "az appservice plan create --name ${APP_SERVICE_NAME}-plan --resource-group $RESOURCE_GROUP --sku B1 --is-linux"
                    bat "az webapp create --resource-group $RESOURCE_GROUP --plan ${APP_SERVICE_NAME}-plan --name $APP_SERVICE_NAME --runtime \"PYTHON:${PYTHON_VERSION}\""
                    bat "az webapp config set --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --startup-file \"gunicorn --bind=0.0.0.0 --timeout 600 app:app\""
                    
                    // Create deployment package
                    bat "powershell Compress-Archive -Path ./* -DestinationPath ./deploy.zip -Force"
                    
                    // Deploy the package
                    bat "az webapp deployment source config-zip --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src ./deploy.zip"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
        always {
            cleanWs()
        }
    }
}
