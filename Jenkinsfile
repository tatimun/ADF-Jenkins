pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                // Clona el repositorio desde la rama 'main'
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/tatimun/ADF-Jenkins.git'
            }
        }

        stage('Install NPM Packages') {
            steps {
                // Instalamos los paquetes de npm que estén en el package.json en la carpeta build
                sh 'npm install --prefix build'
            }
        }

        stage('Validate Data Factory') {
            steps {
                // Autenticamos usando la credencial de Azure Service Principal
                withCredentials([azureServicePrincipal(credentialsId: 'azure-credentials')]) {
                    sh '''
                    npm run --prefix build build validate /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest
                    '''
                }
            }
        }

        stage('Generate ARM Template') {
            steps {
                // Autenticamos usando la credencial de Azure Service Principal
                withCredentials([azureServicePrincipal(credentialsId: 'azure-credentials')]) {
                    sh '''
                    npm run --prefix build build export /subscriptions/${AZURE_SUBSCRIPTION_ID}/resourceGroups/testRG/providers/Microsoft.DataFactory/factories/tatidatatest/factoryId "ArmTemplate"
                    '''
                }
            }
        }

        stage('Publish Artifact') {
            steps {
                // Publicamos el artefacto generado
                archiveArtifacts artifacts: 'build/ArmTemplate/*', allowEmptyArchive: true
            }
        }

        stage('Commit and Push ARM Template') {
            steps {
                script {
                    // Utilizamos credenciales de Git de forma segura
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                        git config --global user.email "apuntatis@gmail.com"
                        git config --global user.name "tatimun"
                        git add build/ArmTemplate/*
                        git commit -m "Exported Data Factory ARM template"
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/tatimun/ADF-Jenkins.git main
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Nos deslogueamos de Azure al final
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
