pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-credentials' // Credenciales de Azure
        GIT_CREDENTIALS_ID = 'github-credentials'  // Credenciales de GitHub
    }
    stages {
        stage('Checkout SCM') {
            steps {
                // Clonar el repositorio de GitHub
                git url: 'https://github.com/tatimun/ADF-Jenkins.git', branch: 'main', credentialsId: "${GIT_CREDENTIALS_ID}"
            }
        }
        stage('Login to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: "${AZURE_CREDENTIALS_ID}")]) {
                    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID'
                }
            }
        }
        stage('Export ARM Template') {
            steps {
                // Exportar el ARM template de Data Factory
                sh '''
                    az datafactory export-arm-template --resource-group testRG \
                        --factory-name tatidatatest \
                        --output-path ./arm-templates/datafactory-template.json
                '''
            }
        }
        stage('Commit ARM Template') {
            steps {
                // Hacer commit del ARM template exportado al repositorio
                sh '''
                    git add ./arm-templates/datafactory-template.json
                    git commit -m "Exported Data Factory ARM template"
                    git push origin main
                '''
            }
        }
    }
    post {
        always {
            sh 'az logout'
        }
    }
}
