pipeline {
    agent any

    environment {
        IR_URL = 'https://app.stage.invisirisk.com'
        IR_TOKEN = credentials('IR_TOKEN')
        DEBUG = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Required Tools') {
            steps {
                sh '''
                    set -e

                    apt-get update
                    apt-get install -y curl ca-certificates nodejs npm
                '''
            }
        }

        stage('Run Parallel Jobs') {
            parallel {
                stage('Job 1 - Install Dependencies') {
                    steps {
                        sh '''
                            set -e

                            echo "================================"
                            echo "InvisiRisk PSE setup"
                            echo "================================"

                            curl -sSf \
                              -H "x-api-key: $IR_TOKEN" \
                              "$IR_URL/ingestionapi/v1/pse/bootstrap" | bash

                            . /tmp/ir_envs

                            echo "================================"
                            echo "Installing dependencies"
                            echo "================================"

                            node --version
                            npm --version
                            npm install --legacy-peer-deps

                            echo "================================"
                            echo "InvisiRisk PSE cleanup"
                            echo "================================"

                            pse-data-collector cleanup || true
                        '''
                    }
                }

                stage('Job 2 - Dependency Check') {
                    steps {
                        sh '''
                            set -e

                            echo "================================"
                            echo "InvisiRisk PSE setup"
                            echo "================================"

                            curl -sSf \
                              -H "x-api-key: $IR_TOKEN" \
                              "$IR_URL/ingestionapi/v1/pse/bootstrap" | bash

                            . /tmp/ir_envs

                            echo "================================"
                            echo "Dependency check"
                            echo "================================"

                            npm install --legacy-peer-deps
                            npm ls || true

                            echo "================================"
                            echo "InvisiRisk PSE cleanup"
                            echo "================================"

                            pse-data-collector cleanup || true
                        '''
                    }
                }

                stage('Job 3 - Test') {
                    steps {
                        sh '''
                            set -e

                            echo "================================"
                            echo "InvisiRisk PSE setup"
                            echo "================================"

                            curl -sSf \
                              -H "x-api-key: $IR_TOKEN" \
                              "$IR_URL/ingestionapi/v1/pse/bootstrap" | bash

                            . /tmp/ir_envs

                            echo "================================"
                            echo "Running tests"
                            echo "================================"

                            npm install --legacy-peer-deps
                            npm test

                            echo "================================"
                            echo "InvisiRisk PSE cleanup"
                            echo "================================"

                            pse-data-collector cleanup || true
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Jenkins pipeline finished.'
        }
    }
}
