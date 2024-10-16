pipeline {
    agent any

    environment {
        // Agregamos el ID de suscripción y otros secretos como variables de entorno
        AZURE_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        AZURE_TENANT_ID = credentials('azure-tenant-id')
        AZURE_CLIENT_ID = credentials('azure-client-id')
        AZURE_CLIENT_SECRET = credentials('azure-client-secret')
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Clona el repositorio de tu código
                git credentialsId: 'github-credentials', url: 'https://github.com/tatimun/ADF-Jenkins.git'
            }
        }

        stage('Install Node.js') {
            steps {
                script {
                    // Instalamos Node.js si no está presente
                    sh '''
                    if ! command -v node &> /dev/null; then
                        curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
                        apt-get install -y nodejs
                    fi
                    '''
                }
            }
        }

        stage('Install NPM Packages') {
            steps {
                // Instalamos los paquetes de npm que estén en el package.json
                sh '''
                npm install --prefix <folder-of-the-package.json-file>
                '''
            }
        }

        stage('Validate Data Factory') {
            steps {
                // Validamos los recursos de Data Factory
                sh '''
                npm run build validate <folder-of-the-package.json-file> /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/<Your-ResourceGroup-Name>/providers/Microsoft.DataFactory/factories/<Your-Factory-Name>
                '''
            }
        }

        stage('Generate ARM Template') {
            steps {
                // Generamos la plantilla ARM en el destino
                sh '''
                npm run build export <folder-of-the-package.json-file> /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/<Your-ResourceGroup-Name>/providers/Microsoft.DataFactory/factories/<Your-Factory-Name> "ArmTemplate"
                '''
            }
        }

        stage('Publish Artifact') {
            steps {
                // Publicamos el artefacto generado
                archiveArtifacts artifacts: '<folder-of-the-package.json-file>/ArmTemplate/*', allowEmptyArchive: true
            }
        }

        stage('Commit and Push ARM Template') {
            steps {
                script {
                    // Configuramos las credenciales de Git para hacer el commit
                    sh '''
                    git config --global user.email "apuntatis@gmail.com"
                    git config --global user.name "tatimun"
                    git add <folder-of-the-package.json-file>/ArmTemplate/*
                    git commit -m "Exported Data Factory ARM template"
                    git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/tatimun/ADF-Jenkins.git main
                    '''
                }
            }
        }
    }

    post {
        always {
            // Nos deslogueamos de Azure siempre al final
            sh 'az logout'
        }
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
    }
}
