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
                        credentialsId: github-credentials
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
                }
            }
        }

        stage('Commit and Push ARM Template') {
            steps {
                script {
                    // Agregar y hacer commit de los ARM templates generados
                    sh '''
                    git config user.email "your-email@example.com"
                    git config user.name "Jenkins"
                    git add .
                    git commit -m "Updated ARM template for Data Factory"
                    '''

                    // Push de los cambios
                    sh '''
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

