pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS_ID = 'nexus-credentials' // ID de las credenciales configuradas
        NEXUS_URL = 'http://localhost:8081' // Cambia esto por la URL de tu Nexus
        NEXUS_REPOSITORY = 'azure-releases' // Cambia esto por el nombre de tu repositorio en Nexus
        ARTIFACT_VERSION = '1.0.0'
        ARTIFACT_ID = 'ArmTemplates'
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
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[
                    artifactId: "${ARTIFACT_ID}", 
                    classifier: '', 
                    file: 'build/ArmTemplate/ARMTemplateForFactory.json', 
                    type: 'json'
                ]], 
                credentialsId: "${NEXUS_CREDENTIALS_ID}",
                groupId: "${GROUP_ID}",
                nexusUrl: "${NEXUS_URL}",
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: "${NEXUS_REPOSITORY}",
                version: "${ARTIFACT_VERSION}"
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
