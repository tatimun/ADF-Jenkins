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

        stage('Debug: List Files') {
            steps {
                sh 'ls -R /var/jenkins_home/workspace/Azure/AzureDataFactory'
            }
        }

        stage('Validate ARM Template') {
            steps {
                script {
                    sh '''
                    node build/node_modules/@microsoft/azure-data-factory-utilities/lib/index validate \
                    /var/jenkins_home/workspace/Azure/AzureDataFactory \
                    /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest
                    '''
                }
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

                    sh '''
                    npm run --prefix build build export \
                    /var/jenkins_home/workspace/Azure/AzureDataFactory \
                    /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest \
                    ArmTemplate
                    '''

                    sh 'cd build/ArmTemplate && zip -r ${FILE_NAME} .'
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

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    passwordVariable: 'NEXUS_PASSWORD',
                    usernameVariable: 'NEXUS_USERNAME')]) {

                    sh """
                    curl -v -u \$NEXUS_USERNAME:\$NEXUS_PASSWORD \
                    --upload-file build/ArmTemplate/${FILE_NAME} \
                    ${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/${ARTIFACT_ID}/${ARTIFACT_VERSION}/${FILE_NAME}
                    """
                }
            }
        }

        stage('Run PowerShell Script') {
            steps {
                powershell '''
                .\\build\\PrePostDeploymentScript.ps1 -armTemplate "ARMTemplateForFactory.json" -ResourceGroupName "TestRG" -DataFactoryName "testtutorialtati" -predeployment $true -deleteDeployment $false
                '''
            }
        }
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
