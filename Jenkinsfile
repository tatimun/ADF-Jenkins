pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-credentials' // El ID de las credenciales de Azure que configuraste en Jenkins
    }
    stages {
        stage('Checkout SCM') {
            steps {
                // Clonar el repositorio de GitHub
                git url: 'https://github.com/tatimun/ADF-Jenkins.git', branch: 'main', credentialsId: 'github-credentials'
            }
        }
        stage('Login to Azure') {
            steps {
                // Usar las credenciales del Service Principal para iniciar sesión en Azure
                withCredentials([azureServicePrincipal(credentialsId: "${AZURE_CREDENTIALS_ID}")]) {
                    sh '''
                        az login --service-principal \
                            --username $AZURE_CLIENT_ID \
                            --password $AZURE_CLIENT_SECRET \
                            --tenant $AZURE_TENANT_ID
                    '''
                }
            }
        }
        stage('List Azure Subscriptions') {
            steps {
                // Listar las suscripciones de Azure
                sh 'az account list --output table'
            }
        }
    }
    post {
        always {
            // Cerrar sesión de Azure después de ejecutar el pipeline
            sh 'az logout'
        }
    }
}

