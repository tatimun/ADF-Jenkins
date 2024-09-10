pipeline {
    agent any

    environment {
        NODE_VERSION = '18.x'
        REPO_PATH = "${env.WORKSPACE}/ADF" // Ajusta con la carpeta correcta dentro de tu repositorio
        SUBSCRIPTION_ID = "${env.SUBSCRIPTION_ID}" // Define en Jenkins o en el entorno
        RESOURCE_GROUP = "${env.RESOURCE_GROUP}" // Define en Jenkins o en el entorno
        DATA_FACTORY_NAME = "tatidatatest" // El nombre de tu Data Factory
    }

    stages {
        stage('Checkout') {
            steps {
                // Clonar el repositorio
                git url: 'https://github.com/tatimun/ADF-Jenkins', branch: 'main'
            }
        }

        stage('Install Node.js') {
            steps {
                sh """
                # Instala Node.js
                curl -sL https://deb.nodesource.com/setup_${NODE_VERSION} | bash -
                apt-get install -y nodejs
                """
            }
        }

        stage('Install npm package') {
            steps {
                dir(REPO_PATH) {
                    sh 'npm install' // Ejecuta npm install dentro del directorio
                }
            }
        }

        stage('Validate Data Factory resources') {
            steps {
                dir(REPO_PATH) {
                    sh """
                    npm run build validate ${env.REPO_PATH}/build /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATA_FACTORY_NAME}
                    """
                }
            }
        }

        stage('Generate ARM Template') {
            steps {
                dir(REPO_PATH) {
                    sh """
                    npm run build export ${env.REPO_PATH}/build /subscriptions/${SUBSCRIPTION_ID}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATA_FACTORY_NAME} "ArmTemplate"
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
