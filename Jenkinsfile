
pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE = 'manairoeya/monapp'
        TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code recupere depuis GitHub'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker version'
                bat 'docker build -t %IMAGE%:%TAG% .'
            }
        }

        stage('Smoke Test') {
            steps {
                bat '''
                    @echo off
                    docker rm -f monapp_test 2>NUL
                    docker run -d --name monapp_test -p 8083:80 %IMAGE%:%TAG%
                    if errorlevel 1 exit /b 1

                    timeout /t 5 /nobreak >NUL

                    curl.exe -f http://localhost:8083/ >NUL
                    if errorlevel 1 exit /b 1

                    echo Application HTTP test successful
                '''
            }
        }

        stage('Push Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        @echo off
                        echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin
                        if errorlevel 1 exit /b 1

                        docker tag %IMAGE%:%TAG% %IMAGE%:latest

                        docker push %IMAGE%:%TAG%
                        if errorlevel 1 exit /b 1

                        docker push %IMAGE%:latest
                        if errorlevel 1 exit /b 1

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            bat(returnStatus: true, script: 'docker rm -f monapp_test')
        }
        success {
            echo 'SUCCESS: Build + Test + Push completed!'
        }
        failure {
            echo 'FAILURE: Check Console Output.'
        }
    }
}
