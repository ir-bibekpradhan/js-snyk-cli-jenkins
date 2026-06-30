pipeline {
    agent any

    environment {
        IR_URL = 'https://app.stage.invisirisk.com'
        IR_TOKEN = credentials('IR_API_KEY')
        DEBUG = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run in Linux Docker Container') {
            steps {
                bat '''
                    docker --version

                    docker run --rm ^
                      -e IR_URL="%IR_URL%" ^
                      -e IR_TOKEN="%IR_TOKEN%" ^
                      -e DEBUG="%DEBUG%" ^
                      -v "%WORKSPACE%:/workspace" ^
                      -w /workspace ^
                      python:3.11 ^
                      bash -lc "set -e; \
                        echo Running inside Linux container; \
                        uname -a; \
                        python --version; \
                        pip --version; \
                        echo Starting InvisiRisk PSE setup; \
                        curl -sSf -H \\"x-api-key: ${IR_TOKEN}\\" \\"${IR_URL}/ingestionapi/v1/pse/bootstrap\\" | bash; \
                        . /tmp/ir_envs; \
                        echo Running user script; \
                        if [ -f requirements.txt ]; then pip install -r requirements.txt; else echo requirements.txt not found, skipping pip install; fi; \
                        pse-data-collector cleanup || true"
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
                echo Jenkins job finished.
                exit /b 0
            '''
        }
    }
}
