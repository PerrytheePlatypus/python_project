pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal-py'
        RESOURCE_GROUP = 'python-webapp-rg'
        APP_SERVICE_NAME = 'python-webapp-service-adi-7'
        PYTHON_VERSION = '3.10'
        // Use a more dynamic way to find Python if possible
        PYTHON_PATH = 'C:\\Users\\91907\\AppData\\Local\\Programs\\Python\\Python310\\python.exe'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/PerrytheePlatypus/python_project.git'
            }
        }
        
        stage('Build') {
            steps {
                bat '''
                    "%PYTHON_PATH%" --version
                    "%PYTHON_PATH%" -m pip install --upgrade pip
                    "%PYTHON_PATH%" -m pip install -r requirements.txt
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    // Login to Azure
                    bat 'az login --service-principal -u "%AZURE_CLIENT_ID%" -p "%AZURE_CLIENT_SECRET%" --tenant "%AZURE_TENANT_ID%"'
                    
                    // Check if resource group exists before creating
                    bat 'az group show --name %RESOURCE_GROUP% || az group create --name %RESOURCE_GROUP% --location eastus'
                    
                    // Create app service plan if it does not exist
                    bat 'az appservice plan show --name %APP_SERVICE_NAME%-plan --resource-group %RESOURCE_GROUP% || az appservice plan create --name %APP_SERVICE_NAME%-plan --resource-group %RESOURCE_GROUP% --sku B1 --is-linux'
                    
                    // Create web app if it does not exist
                    bat 'az webapp show --name %APP_SERVICE_NAME% --resource-group %RESOURCE_GROUP% || az webapp create --resource-group %RESOURCE_GROUP% --plan %APP_SERVICE_NAME%-plan --name %APP_SERVICE_NAME% --runtime "PYTHON|%PYTHON_VERSION%"'
                    
                    // Set startup file
                    bat 'az webapp config set --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 app:app"'
                    
                    // Compress files
                    bat 'powershell Compress-Archive -Path ./* -DestinationPath ./deploy.zip -Force'
                    
                    // Deploy
                    bat 'az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./deploy.zip --timeout 1800'
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
    }
}
