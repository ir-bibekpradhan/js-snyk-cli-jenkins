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

        stage('Check Docker') {
            steps {
                sh '''
                    docker --version
                    docker pull node:20-bookworm
                '''
            }
        }

        stage('Run Parallel Jobs') {
            parallel {
                stage('Job 1 - Install Dependencies') {
                    steps {
                        sh '''
                            docker run --rm \
                              --volumes-from jenkins \
                              -e IR_URL="$IR_URL" \
                              -e IR_TOKEN="$IR_TOKEN" \
                              -e DEBUG="$DEBUG" \
                              -w "$WORKSPACE" \
                              node:20-bookworm \
                              bash -lc '
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
                              '
                        '''
                    }
                }

                stage('Job 2 - Dependency Check') {
                    steps {
                        sh '''
                            docker run --rm \
                              --volumes-from jenkins \
                              -e IR_URL="$IR_URL" \
                              -e IR_TOKEN="$IR_TOKEN" \
                              -e DEBUG="$DEBUG" \
                              -w "$WORKSPACE" \
                              node:20-bookworm \
                              bash -lc '
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
                              '
                        '''
                    }
                }

                stage('Job 3 - Test') {
                    steps {
                        sh '''
                            docker run --rm \
                              --volumes-from jenkins \
                              -e IR_URL="$IR_URL" \
                              -e IR_TOKEN="$IR_TOKEN" \
                              -e DEBUG="$DEBUG" \
                              -w "$WORKSPACE" \
                              node:20-bookworm \
                              bash -lc '
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
                              '
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
