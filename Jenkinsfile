pipeline {

    agent any

    environment {
        APP_GIT_URL = ''
        APP_BRANCH = ''
        APP_NAME = ''
        DEPLOY_PATH = ''
    }

    stages {

        stage('Load ENV File') {

            steps {

                script {

                    def props = readProperties file: 'env/dev.env'

                    env.APP_GIT_URL = props.APP_GIT_URL
                    env.APP_BRANCH = props.APP_BRANCH
                    env.APP_NAME = props.APP_NAME
                    env.DEPLOY_PATH = props.DEPLOY_PATH
                }
            }
        }

        stage('Clone Application Repo') {

            steps {

                dir('application-code') {

                    git branch: "${env.APP_BRANCH}",
                    url: "${env.APP_GIT_URL}"
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
                cp application-code/target/*.war ${env.DEPLOY_PATH}
                """
            }
        }
    }
}
