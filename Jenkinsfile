pipeline {
    agent any

    environment {
        NODE_VERSION = '18.x'
        REPO_PATH = "${env.WORKSPACE}/ADF" // Ajusta la carpeta según tu estructura
        SUBSCRIPTION_ID = "${env.SUBSCRIPTION_ID}"
        RESOURCE_GROUP = "${env.RESOURCE_GROUP}"
        DATA_FACTORY_NAME = "tatidatatest"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/tatimun/ADF-Jenkins', branch: 'main'
            }
        }

        stage('Verify Node.js and npm installation') {
            steps {
                bat """
                node --version
                npm --version
                """
            }
        }

        stage('Install npm packages') {
            steps {
                dir(REPO_PATH) {
                    bat 'npm install' // Ejecuta npm install dentro del directorio
                }
            }
        }

        stage('Validate Data Factory resources') {
            steps {
                dir(REPO_PATH) {
                    bat """
                    npm run build validate ${env.REPO_PATH} /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATA_FACTORY_NAME}
                    """
                }
            }
        }

        stage('Generate ARM Template') {
            steps {
                dir(REPO_PATH) {
                    bat """
                    npm run build export ${env.REPO_PATH} /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATA_FACTORY_NAME} "ArmTemplate"
                    """
                }
            }
        }

        stage('Publish ARM Template Artifact') {
            steps {
                archiveArtifacts artifacts: 'ArmTemplate/**', fingerprint: true
            }
        }
    }

    post {
        always {
            cleanWs() // Limpia el workspace después de la ejecución
        }
    }
}
