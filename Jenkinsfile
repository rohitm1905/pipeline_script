def APP_GIT_URL = ""
def APP_BRANCH = ""
def APP_NAME = ""
def DEPLOY_PATH = ""

pipeline {

    agent eks-node

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

                    APP_GIT_URL = props['APP_GIT_URL']
                    APP_BRANCH = props['APP_BRANCH']
                    APP_NAME = props['APP_NAME']
                    DEPLOY_PATH = props['DEPLOY_PATH']

                    echo "APP_GIT_URL = ${APP_GIT_URL}"
                    echo "APP_BRANCH = ${APP_BRANCH}"
                    echo "APP_NAME = ${APP_NAME}"
                    echo "DEPLOY_PATH = ${DEPLOY_PATH}"
                }
            }
        }

        stage('Clone Application Repo') {

            steps {

                dir('application-code') {

                    git branch: "${APP_BRANCH}",
                    url: "${APP_GIT_URL}"
                }
            }
        }

        stage('Build Application') {

            steps {

                dir('application-code') {

                    sh 'mvn clean package'
                }
            }
        }

        stage('Deploy WAR') {

            steps {

                sh """
                cp application-code/target/*.war ${DEPLOY_PATH}
                """
            }
        }

        stage('Verify Deployment') {

            steps {

                sh """
                ls -lrt ${DEPLOY_PATH}
                """
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
