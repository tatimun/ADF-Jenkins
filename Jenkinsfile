pipeline {
    agent any

    environment {
        NODE_VERSION = '18.x'
        REPO_PATH = "${env.WORKSPACE}/build" // Apunta a la carpeta 'build' donde está el package.json
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
                dir(REPO_PATH) { // Ahora apunta a la carpeta build
                    bat 'npm install'
                }
            }
        }

        stage('Validate Data Factory resources') {
            steps {
                dir(REPO_PATH) {
                    bat """
                    npm run build validate ${REPO_PATH} /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATA_FACTORY_NAME}
                    """
                }
            }
        }

        stage('Generate ARM Template') {
            steps {
                dir(REPO_PATH) {
                    bat """
                    npm run build export ${REPO_PATH} /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATA_FACTORY_NAME} "ArmTemplate"
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
