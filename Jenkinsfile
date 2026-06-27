def APP_GIT_URL = ''
def APP_BRANCH = ''
def PROJECT_LIST = ''
def BUILD_DIR = ''

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

                    APP_GIT_URL = props['APP_GIT_URL']
                    APP_BRANCH = props['APP_BRANCH']
                    PROJECT_LIST = props['PROJECT_LIST'] ?: ''

                    BUILD_DIR = "build-${env.BUILD_NUMBER}"

                    echo "APP_GIT_URL = ${APP_GIT_URL}"
                    echo "APP_BRANCH = ${APP_BRANCH}"
                    echo "PROJECT_LIST = ${PROJECT_LIST}"
                    echo "BUILD_DIR = ${BUILD_DIR}"
                }
            }
        }

        stage('Clone Application Repo') {

            steps {

                script {

                    if (!APP_GIT_URL) {
                        error 'APP_GIT_URL is not set in env/dev.env'
                    }

                    sh "mkdir -p ${BUILD_DIR}"

                    dir(BUILD_DIR) {

                        git branch: APP_BRANCH,
                            url: APP_GIT_URL

                        echo "Repository cloned successfully"

                        sh 'pwd'
                        sh 'ls -lrt'
                    }
                }
            }
        }

        stage('Build Projects') {

            steps {

                script {

                    def projects = PROJECT_LIST.tokenize(',')
                                               .collect { it.trim() }
                                               .findAll { it }

                    if (!projects) {
                        error 'PROJECT_LIST is empty. Set PROJECT_LIST in env/dev.env.'
                    }

                    echo "Projects to Build : ${projects}"

                    projects.each { proj ->

                        dir("${BUILD_DIR}/${proj}") {

                            echo "======================================"
                            echo "Building Project : ${proj}"
                            echo "======================================"

                            sh 'pwd'
                            sh 'ls -lrt'

                            sh 'mvn clean package'

                            echo "Build Completed Successfully for ${proj}"
                        }

                    }

                }

            }

        }

    }

    post {

        success {

            echo "======================================"
            echo "Pipeline Completed Successfully"
            echo "======================================"

        }

        failure {

            echo "======================================"
            echo "Pipeline Failed"
            echo "======================================"

        }

    }

}
