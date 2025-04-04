pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal-py'
        RESOURCE_GROUP = 'python-webapp-rg'
        APP_SERVICE_NAME = 'python-webapp-service-adi-7'
        PYTHON_VERSION = '3.10'
        PYTHON_PATH = 'C:\\Users\\91907\\AppData\\Local\\Programs\\Python\\Python310\\python.exe'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "Checking out code from GitHub repository..."
                }
                git branch: 'main', url: 'https://github.com/PerrytheePlatypus/python_project.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo "Building the application..."
                }
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
                    bat '''
                        echo "Logging in to Azure..."
                        az login --service-principal -u "%AZURE_CLIENT_ID%" -p "%AZURE_CLIENT_SECRET%" --tenant "%AZURE_TENANT_ID%" --verbose
                        echo "Checking if resource group exists..."
                        az group show --name %RESOURCE_GROUP% --output json --verbose || az group create --name %RESOURCE_GROUP% --location eastus --verbose
                        echo "Checking if app service plan exists..."
                        az appservice plan show --name %APP_SERVICE_NAME%-plan --resource-group %RESOURCE_GROUP% --output json --verbose || az appservice plan create --name %APP_SERVICE_NAME%-plan --resource-group %RESOURCE_GROUP% --sku B1 --is-linux --verbose
                        echo "Checking if web app exists..."
                        az webapp show --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --output json --verbose || az webapp create --resource-group %RESOURCE_GROUP% --plan %APP_SERVICE_NAME%-plan --name %APP_SERVICE_NAME% --runtime "PYTHON|%PYTHON_VERSION%" --verbose
                        echo "Setting web app config..."
                        az webapp config set --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 app:app" --verbose
                        echo "Compressing files..."
                        powershell Compress-Archive -Path ./* -DestinationPath ./deploy.zip -Force
                        echo "Deploying web app..."
                        az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./deploy.zip --timeout 1800 --verbose
                    '''
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
