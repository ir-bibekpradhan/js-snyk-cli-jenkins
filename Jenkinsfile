pipeline {
    agent any

    environment {
        IR_URL = 'https://app.stage.invisirisk.com'
        IR_TOKEN = credentials('IR_API_KEY')
        DEBUG = 'true'
        DOCKER_BIN = 'C:\\Program Files\\Docker\\Docker\\resources\\bin'
        DOCKER_EXE = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
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
                    echo Adding Docker Desktop bin folder to PATH
                    set "PATH=%DOCKER_BIN%;%PATH%"

                    echo Checking Docker
                    "%DOCKER_EXE%" --version

                    echo Checking Docker credential helper
                    where docker-credential-desktop

                    echo Pulling Node image
                    "%DOCKER_EXE%" pull node:20-bookworm

                    echo Running build inside Linux Node container
                    "%DOCKER_EXE%" run --rm ^
                      -e IR_URL="%IR_URL%" ^
                      -e IR_TOKEN="%IR_TOKEN%" ^
                      -e DEBUG="%DEBUG%" ^
                      -v "%WORKSPACE%:/workspace" ^
                      -w /workspace ^
                      node:20-bookworm ^
                      bash -lc "set -e; \
                        echo Running inside Linux Node container; \
                        uname -a; \
                        node --version; \
                        npm --version; \
                        echo Starting InvisiRisk PSE setup; \
                        curl -sSf -H \\"x-api-key: ${IR_TOKEN}\\" \\"${IR_URL}/ingestionapi/v1/pse/bootstrap\\" | bash; \
                        . /tmp/ir_envs; \
                        echo Installing npm dependencies; \
                        if [ -f package.json ]; then npm install --legacy-peer-deps; else echo package.json not found; exit 1; fi; \
                        echo Running dependency check; \
                        npm ls || true; \
                        pse-data-collector cleanup || true"
                '''
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
