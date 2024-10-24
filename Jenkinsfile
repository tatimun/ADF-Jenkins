pipeline {
    agent any

    environment {
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'
        NEXUS_URL = 'http://nexus:8081'
        NEXUS_REPOSITORY = 'arm-templates'
        ARTIFACT_ID = 'ArmTemplates'
        FILE_NAME = 'armtemplates.zip'
        BASE_VERSION = '1.0'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/tatimun/ADF-Jenkins.git',
                        credentialsId: 'github-credentials',
                        relativeTargetDir: 'build'
                    ]]
                ])
            }
        }

        stage('Debug: Print Working Directory and List Build Folder') {
            steps {
                bat 'echo Current Directory: %cd%'
                bat 'dir build'
            }
        }

        stage('Install NPM Packages') {
            steps {
                bat '''
                cd build
                npm install
                '''
            }
        }

        stage('Validate ARM Template') {
            steps {
                bat '''
                cd build
                node node_modules\\@microsoft\\azure-data-factory-utilities\\lib\\index validate %WORKSPACE%\\build /subscriptions/%AZURE_SUBSCRIPTION_ID%/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest
                '''
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

                    bat '''
                    cd build
                    npm run build export ^
                    %WORKSPACE%\\build ^
                    /subscriptions/%AZURE_SUBSCRIPTION_ID%/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest ^
                    ArmTemplate
                    '''

                    bat '''
                    cd ArmTemplate
                    "C:\\Program Files\\7-Zip\\7z.exe" a -tzip ${FILE_NAME} .
                    '''
                }
            }
        }

        stage('Increment Version') {
            steps {
                script {
                    def buildNumber = currentBuild.number
                    env.ARTIFACT_VERSION = "${BASE_VERSION}.${buildNumber}"
                    echo "New artifact version: ${ARTIFACT_VERSION}"
                }
            }
        }

        stage('Print Artifact Version') {
            steps {
                script {
                    echo "----------------------------------------------------"
                    echo "The current artifact version is: ${ARTIFACT_VERSION}"
                    echo "----------------------------------------------------"
                }
            }
        }

    }

}

