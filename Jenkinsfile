pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS_ID = 'nexus-credentials' // ID de las credenciales configuradas en Jenkins para Nexus
        NEXUS_URL = 'http://nexus:8081' // Cambia por la URL de tu Nexus
        NEXUS_REPOSITORY = 'arm-templates' // Nombre del repositorio Raw en Nexus
        ARTIFACT_ID = 'ArmTemplates'
        FILE_NAME = 'armtemplates.zip' // El nombre del archivo que vas a subir
        BASE_VERSION = '1.0' // Base de la versión (ej. 1.0)
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

        stage('Increment Version') {
            steps {
                script {
                    // Obtener el número de build actual como parte del incremento
                    def buildNumber = currentBuild.number

                    // Actualizar el ARTIFACT_VERSION usando el número de build
                    env.ARTIFACT_VERSION = "${BASE_VERSION}.${buildNumber}"
                    echo "New artifact version: ${ARTIFACT_VERSION}"
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

        stage('Pre Deployment - Azure PowerShell') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'azure-credentials', 
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                    tenantIdVariable: 'AZURE_TENANT_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET'
                )]) {
                    // Comando PowerShell para ejecutar el script
                    sh '''
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                    az account set --subscription $AZURE_SUBSCRIPTION_ID

                    pwsh -File ./build/Job/PrePostDeploymentScript.ps1 `
                        -armTemplate "./build/Job/ARMTemplateForFactory.json" `
                        -ResourceGroupName 'TestRG' `
                        -DataFactoryName 'tatidatatest' `
                        -predeployment $true `
                        -deleteDeployment $false
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                // Verifica si hay cuentas activas de Azure antes de intentar desconectar
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

