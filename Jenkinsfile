pipeline {
    agent any

    environment {
        NODE_VERSION = '18.x'
        AZURE_SUBSCRIPTION = 'subscrizzzzz
        RESOURCE_GROUP = 'testRG'
        DATAFACTORY_NAME = 'tatidatatest'
        PACKAGE_FOLDER = 'package.json'
        OUTPUT_FOLDER = 'ArmTemplate'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Node.js') {
            steps {
                script {
                    // Instalar Node.js 18.x
                    sh "curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -"
                    sh "sudo apt-get install -y nodejs"
                }
            }
        }

        stage('Install NPM Packages') {
            steps {
                // Instalar las dependencias desde el package.json
                dir("${PACKAGE_FOLDER}") {
                    sh 'npm install'
                }
            }
        }

        stage('Validate Data Factory') {
            steps {
                // Validar los recursos de Data Factory
                dir("${PACKAGE_FOLDER}") {
                    sh "npm run build validate ${WORKSPACE}/${PACKAGE_FOLDER} /subscriptions/${AZURE_SUBSCRIPTION}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATAFACTORY_NAME}"
                }
            }
        }

        stage('Generate ARM Template') {
            steps {
                // Exportar la plantilla ARM
                dir("${PACKAGE_FOLDER}") {
                    sh "npm run build export ${WORKSPACE}/${PACKAGE_FOLDER} /subscriptions/${AZURE_SUBSCRIPTION}/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.DataFactory/factories/${DATAFACTORY_NAME} '${OUTPUT_FOLDER}'"
                }
            }
        }

        stage('Publish Artifact') {
            steps {
                script {
                    // Copiar la plantilla ARM generada a una carpeta del workspace
                    sh "mkdir -p ${WORKSPACE}/arm-templates"
                    sh "cp -r ${WORKSPACE}/${PACKAGE_FOLDER}/${OUTPUT_FOLDER}/* ${WORKSPACE}/arm-templates/"
                }
            }
        }

        stage('Commit and Push ARM Template') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    // Agregar, commitear y hacer push del template ARM al repositorio
                    sh '''
                    git config --global user.email "apuntatis@gmail.com"
                    git config --global user.name "tatimun"
                    git add ${WORKSPACE}/arm-templates/*
                    git commit -m "Exported Data Factory ARM template"
                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tatimun/ADF-Jenkins.git main
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                // Limpiar recursos o cualquier acción post-pipeline
                sh 'az logout'
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
