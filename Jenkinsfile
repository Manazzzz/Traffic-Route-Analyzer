def runCmd(String windowsCmd, String unixCmd = null) {
    if (isUnix()) {
        sh(unixCmd ?: windowsCmd)
    } else {
        bat(windowsCmd)
    }
}

pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        APP_PORT = '8501'
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker compose build || docker-compose build'
                    } else {
                        bat 'docker compose build || docker-compose build'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker compose run --rm --no-deps app python -m pytest tests/ || docker-compose run --rm --no-deps app python -m pytest tests/'
                    } else {
                        bat 'docker compose run --rm --no-deps app python -m pytest tests/ || docker-compose run --rm --no-deps app python -m pytest tests/'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                        docker compose down || docker-compose down
                        docker compose up -d || docker-compose up -d
                        sleep 30
                        '''
                    } else {
                        bat '''
                        docker compose down || docker-compose down
                        docker compose up -d || docker-compose up -d
                        timeout /t 30
                        '''
                    }
                }
            }
        }

        stage('Verify') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                        docker ps
                        '''
                    } else {
                        bat '''
                        docker ps
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'PIPELINE SUCCESS'
            echo 'App: http://localhost:8501'
        }
        failure {
            echo 'PIPELINE FAILED'
        }
    }
}