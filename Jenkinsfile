pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                // Clona el repositorio desde GitHub
                git branch: 'main', credentialsId: 'GITHUB_CREDENTIALS', url: 'https://github.com/tatimun/ADF-Jenkins.git'
            }
        }

        stage('Install NPM Packages') {
            steps {
                // Instalamos las dependencias de npm
                sh 'npm install --prefix build'
            }
        }

        stage('Validate Data Factory') {
            steps {
                // Validamos la infraestructura de Data Factory
                withCredentials([azureServicePrincipal(credentialsId: 'azure-credentials')]) {
                    sh '''
                    npm run --prefix build build validate /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest --factoryId tatidatatest
                    '''
                }
            }
        }

        stage('Export ARM Template') {
            steps {
                // Exportamos la plantilla ARM de Data Factory
                withCredentials([azureServicePrincipal(credentialsId: 'azure-credentials')]) {
                    sh '''
                    npm run --prefix build build export /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest --factoryId tatidatatest ArmTemplate
                    '''
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                // Desplegamos la plantilla ARM exportada al entorno de producción
                withCredentials([azureServicePrincipal(credentialsId: 'azure-credentials')]) {
                    sh '''
                    az group deployment create --resource-group prodRG --template-file build/ArmTemplate/ARMTemplate.json --parameters @build/ArmTemplate/parameters.json
                    '''
                }
            }
        }

        stage('Commit and Push Changes to GitHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                        git config --global user.email "apuntatis@gmail.com"
                        git config --global user.name "tatimun"
                        git add build/ArmTemplate/*
                        git commit -m "Updated ARM template for Data Factory"
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/tatimun/ADF-Jenkins.git main
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Cierra sesión de Azure al final
            sh 'az logout || echo "No active session to logout"'
        }
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
    }
}
