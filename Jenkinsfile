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
                    def projects = []
                    if (PROJECT_LIST?.trim()) {
                        projects = PROJECT_LIST.tokenize(',').collect { it.trim() }.findAll { it }
                    } else {
                        def projectOutput = sh(script: """
                            find . -maxdepth 3 -name pom.xml \
                                | grep -v '^./pom.xml$' \
                                | sed 's#^./##' \
                                | sed 's#/pom.xml$##' \
                                | sort -u
                        """, returnStdout: true).trim()
                        projects = projectOutput.tokenize('\n').findAll { it }
                    }

                    if (!projects) {
                        error 'No Maven projects found to build. Define PROJECT_LIST in env/dev.env or ensure subfolders contain pom.xml.'
                    }

                    echo "Projects to build: ${projects}"
                    projects.each { proj ->
                        dir(proj) {
                            echo "Building project ${proj}"
                            sh 'mvn clean package'
                        }
                    }
                }
            }
        }

        stage('Deploy WARs') {
            steps {
                script {
                    def projects = PROJECT_LIST?.trim() ? PROJECT_LIST.tokenize(',').collect { it.trim() }.findAll { it } : sh(script: """
                            find . -maxdepth 3 -name pom.xml \
                                | grep -v '^./pom.xml$' \
                                | sed 's#^./##' \
                                | sed 's#/pom.xml$##' \
                                | sort -u
                        """, returnStdout: true).trim().tokenize('\n').findAll { it }

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
