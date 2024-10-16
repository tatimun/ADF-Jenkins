pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-credentials' // Credenciales de Azure
        GIT_CREDENTIALS_ID = 'github-credentials'  // Credenciales de GitHub
        RESOURCE_GROUP = 'testRG'                  // Reemplaza con el nombre real de tu resource group
        DATA_FACTORY = 'tatidatatest'              // Reemplaza con el nombre real de tu Data Factory
        SUBSCRIPTION_ID = '<tu-subscription-id>'   // Tu ID de suscripción de Azure
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
        stage('Export ARM Template via REST API') {
            steps {
                // Obtener el token de acceso
                script {
                    def accessToken = sh(
                        script: 'az account get-access-token --query accessToken --output tsv',
                        returnStdout: true
                    ).trim()

                    // Llamar a la API REST de Azure para exportar el template
                    sh """
                    curl -X POST \
                    -H "Authorization: Bearer ${accessToken}" \
                    -H "Content-Type: application/json" \
                    https://management.azure.com/subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATA_FACTORY}/exportTemplate?api-version=2018-06-01 \
                    -d '{}' -o ./arm-templates/datafactory-template.json
                    """
                }
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
            sh 'az logout || true'
        }
    }
}
