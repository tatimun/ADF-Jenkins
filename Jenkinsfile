pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/tatimun/ADF-Jenkins.git',
                        credentialsId: 'github-credentials' // Usamos el ID que tienes configurado
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
                    credentialsId: 'azure-credentials' // Usamos el ID de credenciales de Azure
                )]) {
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

        stage('Commit and Push ARM Template') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_TOKEN')]) {
                    // Agregar y hacer commit de los ARM templates generados
                    sh '''
                    git config user.email "apuntatis@gmail.com"
                    git config user.name "tatimun"
                    git add .
                    git commit -m "Updated ARM template for Data Factory"
                    '''

                    // Push de los cambios con credenciales
                    sh '''
                    git pull origin main --rebase
                    git push https://$GITHUB_USERNAME:$GITHUB_TOKEN@github.com/tatimun/ADF-Jenkins.git main
                    '''
                }
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
