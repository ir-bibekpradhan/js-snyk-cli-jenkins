pipeline {
    agent any

    environment {
        IR_URL = 'https://app.stage.invisirisk.com'
        IR_TOKEN = credentials('IR_TOKEN')
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

        stage('Run Parallel Jobs') {
            parallel {
                stage('Job 1 - Install Dependencies') {
                    steps {
                        bat '''
                            set "PATH=%DOCKER_BIN%;%PATH%"

                            "%DOCKER_EXE%" run --rm ^
                              -e IR_URL="%IR_URL%" ^
                              -e IR_TOKEN="%IR_TOKEN%" ^
                              -e DEBUG="%DEBUG%" ^
                              -v "%WORKSPACE%:/workspace" ^
                              -w /workspace ^
                              node:20-bookworm ^
                              bash -lc "set -e; \
                                echo InvisiRisk PSE setup; \
                                curl -sSf -H \\"x-api-key: $IR_TOKEN\\" \\"$IR_URL/ingestionapi/v1/pse/bootstrap\\" | bash; \
                                . /tmp/ir_envs; \
                                node --version; \
                                npm --version; \
                                npm install --legacy-peer-deps; \
                                pse-data-collector cleanup || true"
                        '''
                    }
                }

                stage('Job 2 - Dependency Check') {
                    steps {
                        bat '''
                            set "PATH=%DOCKER_BIN%;%PATH%"

                            "%DOCKER_EXE%" run --rm ^
                              -e IR_URL="%IR_URL%" ^
                              -e IR_TOKEN="%IR_TOKEN%" ^
                              -e DEBUG="%DEBUG%" ^
                              -v "%WORKSPACE%:/workspace" ^
                              -w /workspace ^
                              node:20-bookworm ^
                              bash -lc "set -e; \
                                echo InvisiRisk PSE setup; \
                                curl -sSf -H \\"x-api-key: $IR_TOKEN\\" \\"$IR_URL/ingestionapi/v1/pse/bootstrap\\" | bash; \
                                . /tmp/ir_envs; \
                                npm install --legacy-peer-deps; \
                                npm ls || true; \
                                pse-data-collector cleanup || true"
                        '''
                    }
                }

                stage('Job 3 - Test') {
                    steps {
                        bat '''
                            set "PATH=%DOCKER_BIN%;%PATH%"

                            "%DOCKER_EXE%" run --rm ^
                              -e IR_URL="%IR_URL%" ^
                              -e IR_TOKEN="%IR_TOKEN%" ^
                              -e DEBUG="%DEBUG%" ^
                              -v "%WORKSPACE%:/workspace" ^
                              -w /workspace ^
                              node:20-bookworm ^
                              bash -lc "set -e; \
                                echo InvisiRisk PSE setup; \
                                curl -sSf -H \\"x-api-key: $IR_TOKEN\\" \\"$IR_URL/ingestionapi/v1/pse/bootstrap\\" | bash; \
                                . /tmp/ir_envs; \
                                npm install --legacy-peer-deps; \
                                npm test; \
                                pse-data-collector cleanup || true"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            bat '''
                echo Jenkins pipeline finished.
                exit /b 0
            '''
        }
    }
}
