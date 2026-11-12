pipeline {
    agent any
    options { timestamps() }

    environment {
        IMAGE = 'issaouinour/monapp'
        TAG = "build-${env.BUILD_NUMBER}"
        DOCKER = 'C:\\Users\\nouri\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                bat '"%DOCKER%" version'
                bat '"%DOCKER%" build -t %IMAGE%:%TAG% .'
            }
        }

        stage('Smoke Test') {
            steps {
                bat """
                    "%DOCKER%" run -d --name monapp_test -p 8081:80 %IMAGE%:%TAG%
                    ping -n 3 127.0.0.1 > nul
                    curl -I http://localhost:8081
                    "%DOCKER%" rm -f monapp_test
                """
            }
        }

        stage('Push (Docker Hub)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    bat """
                        echo %PASS% | "%DOCKER%" login -u %USER% --password-stdin
                        "%DOCKER%" tag %IMAGE%:%TAG% %IMAGE%:latest
                        "%DOCKER%" push %IMAGE%:%TAG%
                        "%DOCKER%" push %IMAGE%:latest
                    """
                }
            }
        }