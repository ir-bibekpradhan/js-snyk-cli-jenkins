pipeline {
    agent any

    environment {
        IR_URL = 'https://app.stage.invisirisk.com'
        DEBUG = 'true'
        IR_TOKEN = credentials('IR_API_KEY')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('InvisiRisk PSE Setup') {
            steps {
                bat '''
                    echo Downloading InvisiRisk PSE bootstrap

                    curl -sSf -H "x-api-key: %IR_TOKEN%" "%IR_URL%/ingestionapi/v1/pse/bootstrap?mode=native&runner=any" -o "%WORKSPACE%\\bootstrap.sh"

                    if not exist "%WORKSPACE%\\bootstrap.sh" (
                      echo bootstrap download failed
                      exit /b 1
                    )

                    bash "%WORKSPACE%\\bootstrap.sh"

                    if not exist "C:\\tmp\\ir_envs" (
                      echo C:\\tmp\\ir_envs missing
                      if exist "C:\\tmp\\bootstrap.log" type "C:\\tmp\\bootstrap.log"
                      exit /b 1
                    )
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    node --version
                    npm --version
                    npm install --legacy-peer-deps
                '''
            }
        }

        stage('Dependency Check') {
            steps {
                bat '''
                    npm ls
                    exit /b 0
                '''
            }
        }

        stage('Docker Build') {
            steps {
                bat '''
                    docker --version

                    if exist Dockerfile (
                      docker build -t js-snyk-cli-jenkins:latest .
                    ) else (
                      echo Dockerfile not found, skipping Docker build
                    )
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                powershell '''
                    New-Item -ItemType Directory -Force -Path "jenkins-artifacts" | Out-Null

                    if (Test-Path "jenkins-artifacts\\archive.zip") {
                        Remove-Item "jenkins-artifacts\\archive.zip" -Force
                    }

                    Compress-Archive -Path * -DestinationPath "jenkins-artifacts\\archive.zip" -Force
                '''

                archiveArtifacts artifacts: 'jenkins-artifacts/archive.zip', fingerprint: true
            }
        }
    }

    post {
        always {
            bat '''
                where pse-data-collector >nul 2>nul
                if %ERRORLEVEL% EQU 0 (
                  pse-data-collector cleanup
                ) else (
                  echo pse-data-collector not found, skipping cleanup
                )
                exit /b 0
            '''
        }
    }
}
