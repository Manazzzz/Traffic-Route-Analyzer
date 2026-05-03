def runCmd(String windowsCmd, String unixCmd = null) {
    if (isUnix()) {
        sh(unixCmd ?: windowsCmd)
    } else {
        bat(windowsCmd)
    }
}

def runCompose(String args) {
    if (isUnix()) {
        sh "docker compose ${args} || docker-compose ${args}"
    } else {
        bat "docker compose ${args} || docker-compose ${args}"
    }
}

def cleanupContainers() {
    if (isUnix()) {
        sh '''
        docker rm -f ngd-neo4j ngd-cassandra ngd-app 2>/dev/null || true
        '''
    } else {
        bat '''
        docker rm -f ngd-neo4j ngd-cassandra ngd-app >nul 2>&1
        '''
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
                    runCompose('build --pull')
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // 🔥 skip entrypoint → prevents DB wait
                    runCompose('run --rm --no-deps --entrypoint "" app python -m pytest tests/ -v')
                }
            }
            post {
                always {
                    runCompose('down --remove-orphans')
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    cleanupContainers()

                    if (isUnix()) {
                        sh '''
                        docker compose down --remove-orphans || docker-compose down --remove-orphans
                        docker compose up -d || docker-compose up -d
                        sleep 40
                        '''
                    } else {
                        bat '''
                        docker compose down --remove-orphans || docker-compose down --remove-orphans
                        docker compose up -d || docker-compose up -d
                        timeout /t 40
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
                        echo "Running containers:"
                        docker ps

                        echo "Compose status:"
                        docker compose ps || docker-compose ps
                        '''
                    } else {
                        bat '''
                        echo Running containers:
                        docker ps

                        echo Compose status:
                        docker compose ps || docker-compose ps
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'PIPELINE SUCCESS ✅'
            echo 'App: http://localhost:8501'
        }
        failure {
            echo 'PIPELINE FAILED ❌ - check logs above'
            script {
                runCompose('logs --no-color')
            }
        }
        always {
            echo 'Pipeline finished.'
        }
    }
}