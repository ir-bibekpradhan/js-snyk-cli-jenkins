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

                    set "BOOTSTRAP_SH=%WORKSPACE%\\bootstrap.sh"
                    set "BOOTSTRAP_SH=%BOOTSTRAP_SH:\\=/%"

                    echo Running bootstrap with Git Bash:
                    echo %BOOTSTRAP_SH%

                    bash "%BOOTSTRAP_SH%"

                    bash -lc "test -f /tmp/ir_envs"

                    if %ERRORLEVEL% NEQ 0 (
                      echo /tmp/ir_envs missing inside Git Bash
                      bash -lc "cat /tmp/bootstrap.log 2>/dev/null || true"
                      exit /b 1
                    )
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    bash -lc "source /tmp/ir_envs && node --version && npm --version && npm install --legacy-peer-deps"
                '''
            }
        }

        stage('Dependency Check') {
            steps {
                bat '''
                    bash -lc "source /tmp/ir_envs && npm ls || true"
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
                bash -lc "if command -v pse-data-collector >/dev/null 2>&1; then pse-data-collector cleanup || true; else echo pse-data-collector not found, skipping cleanup; fi"
                exit /b 0
            '''
        }
    }
}
