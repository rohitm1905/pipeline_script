def APP_GIT_URL = ''
def APP_BRANCH = ''
def DEPLOY_PATH = ''
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
                    DEPLOY_PATH = props['DEPLOY_PATH']
                    PROJECT_LIST = props['PROJECT_LIST'] ?: ''
                    SONAR_HOST_URL = props['SONAR_HOST_URL'] ?: ''
                    SONAR_TOKEN = props['SONAR_TOKEN'] ?: ''
                    SONAR_PROJECT_PREFIX = props['SONAR_PROJECT_PREFIX'] ?: ''
                    BUILD_DIR = "build-${env.BUILD_NUMBER}"

                    echo "APP_GIT_URL = ${APP_GIT_URL}"
                    echo "APP_BRANCH = ${APP_BRANCH}"
                    echo "DEPLOY_PATH = ${DEPLOY_PATH}"
                    echo "PROJECT_LIST = ${PROJECT_LIST}"
                    echo "SONAR_HOST_URL = ${SONAR_HOST_URL ? '****' : ''}"
                    echo "SONAR_PROJECT_PREFIX = ${SONAR_PROJECT_PREFIX}"
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
                        git branch: APP_BRANCH, url: APP_GIT_URL
                        sh 'echo "Cloned repo into build folder:"'
                        sh 'pwd'
                        sh 'ls -lart'
                    }
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
                        dir("${BUILD_DIR}/${proj}") {
                            echo "Building project ${proj}"
                            sh 'pwd'
                            sh 'ls -lart'
                            sh 'mvn clean package'

                            // Run Sonar scan if configured
                            def sonarKey = SONAR_PROJECT_PREFIX?.trim() ? "${SONAR_PROJECT_PREFIX}" : proj
                            if (SONAR_HOST_URL?.trim() && SONAR_TOKEN?.trim()) {
                                echo "Running Sonar scan for ${proj} (project key: ${sonarKey})"
                                sh "mvn sonar:sonar -Dsonar.projectKey=${sonarKey} -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_TOKEN}"
                            } else {
                                echo "Sonar variables not set; skipping Sonar scan for ${proj}"
                            }
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
                        sh "mkdir -p ${DEPLOY_PATH}/${proj} && cp ${BUILD_DIR}/${proj}/target/*.war ${DEPLOY_PATH}/${proj}/"
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "ls -lart ${DEPLOY_PATH}"
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
