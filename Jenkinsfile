pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'
        NEXUS_URL = 'http://nexus:8081'
        NEXUS_REPOSITORY = 'arm-templates'
        ARTIFACT_ID = 'ArmTemplates'
        FILE_NAME = 'armtemplates.zip'
        BASE_VERSION = '1.0'
        AZURE_SUBSCRIPTION_ID = 'your_subscription_id' // Reemplaza con tu ID de suscripción
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/tatimun/ADF-Jenkins.git',
                        credentialsId: 'github-credentials'
                    ]]
                ])
            }
        }

        stage('Install NPM Packages') {
            steps {
                sh 'npm install --prefix build'
            }
        }

        stage('Validate ARM Template') {
            steps {
                script {
                    // Ejecutar el comando de validación
                    sh """
                    node build/node_modules/@microsoft/azure-data-factory-utilities/lib/index validate /var/jenkins_home/workspace/Azure/AzureDataFactory /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest
                    """
                }
            }
        }

        // Aquí puedes agregar más etapas como "Generate ARM Template", "Increment Version", etc.
    }

    post {
        always {
            script {
                def result = sh(script: 'az account show', returnStatus: true)
                if (result == 0) {
                    sh 'az logout'
                } else {
                    echo 'No Azure accounts were logged in.'
                }
            }
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
