def DEPLOY_PATH = ''
def PROJECT_LIST = ''

pipeline {
    agent any

    tools {
        maven 'MVN'
    }

    stages {
        stage('Load ENV File') {
            steps {
                echo 'Checking Workspace'
                sh 'pwd'
                sh 'ls -lrt'
                sh 'ls -lrt env'

                script {
                    def props = readProperties file: 'env/dev.env'
                    echo "Properties Loaded: ${props}"

                    DEPLOY_PATH = props['DEPLOY_PATH']
                    PROJECT_LIST = props['PROJECT_LIST'] ?: ''

                    echo "DEPLOY_PATH = ${DEPLOY_PATH}"
                    echo "PROJECT_LIST = ${PROJECT_LIST}"
                }
            }
        }

        stage('Build Projects') {
            steps {
                script {
                    def projects = PROJECT_LIST.tokenize(',').collect { it.trim() }.findAll { it }

                    if (!projects) {
                        error 'PROJECT_LIST is empty. Set PROJECT_LIST in env/dev.env to comma-separated project folder names.'
                    }

                    echo "Projects to build: ${projects}"
                    projects.each { proj ->
                        dir(proj) {
                            echo "Building project ${proj}"
                            sh 'ls -lrt'
                            sh 'mvn clean package'
                        }
                    }
                }
            }
        }

        stage('Deploy WARs') {
            steps {
                script {
                    def projects = PROJECT_LIST.tokenize(',').collect { it.trim() }.findAll { it }

                    projects.each { proj ->
                        echo "Deploying WAR for ${proj}"
                        sh "mkdir -p ${DEPLOY_PATH}/${proj} && cp ${proj}/target/*.war ${DEPLOY_PATH}/${proj}/"
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "ls -lrt ${DEPLOY_PATH}"
            }
        }
    }

    post {
        success {
            echo 'Pipeline Completed Successfully'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
