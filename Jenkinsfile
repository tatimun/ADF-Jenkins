pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS_ID = 'nexus-credentials' // ID de las credenciales configuradas en Jenkins para Nexus
        NEXUS_URL = 'http://localhost:8081' // Cambia por la URL de tu Nexus
        NEXUS_REPOSITORY = 'arm-templates' // Nombre del repositorio Raw en Nexus
        ARTIFACT_VERSION = '1.0.0'
        ARTIFACT_ID = 'ArmTemplates'
        FILE_NAME = 'armtemplates.zip' // El nombre del archivo que vas a subir
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

        stage('Generate ARM Template') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'azure-credentials', 
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID', 
                    tenantIdVariable: 'AZURE_TENANT_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID', 
                    clientSecretVariable: 'AZURE_CLIENT_SECRET')]) {
                    
                    // Generar el ARM template
                    sh '''
                    npm run --prefix build build export \
                    /var/jenkins_home/workspace/Azure/DataFactory \
                    /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest \
                    ArmTemplate
                    '''

                    // Crear el archivo ZIP para subirlo
                    sh 'cd build/ArmTemplate && zip -r ${FILE_NAME} .'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.NEXUS_CREDENTIALS_ID, 
                    passwordVariable: 'NEXUS_PASSWORD', 
                    usernameVariable: 'NEXUS_USERNAME')]) {
                    
                    sh """
                    curl -v -u $NEXUS_USERNAME:$NEXUS_PASSWORD \
                    --upload-file build/ArmTemplate/${FILE_NAME} \
                    ${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/${ARTIFACT_ID}/${ARTIFACT_VERSION}/${FILE_NAME}
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'az logout'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
