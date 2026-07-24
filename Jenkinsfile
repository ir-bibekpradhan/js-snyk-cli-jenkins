pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        IR_URL = 'https://app.dev.invisirisk.com'
        IR_TOKEN = credentials('IR_TOKEN')
        DEBUG = 'true'
    }

    stages {
        stage('Verify Environment') {
            steps {
                sh '''
                    set -e

                    echo "Checking required tools"
                    curl --version
                    node --version
                    npm --version
                '''
            }
        }

        stage('Run Parallel Jobs') {
            failFast true

            parallel {
                stage('Job 1 - Install Dependencies') {
                    steps {
                        dir('job-install') {
                            deleteDir()
                            checkout scm

                            sh '''
                                set -e

                                echo "================================"
                                echo "Job 1: InvisiRisk PSE setup"
                                echo "================================"

                                curl -sSf \
                                  -H "x-api-key: $IR_TOKEN" \
                                  "$IR_URL/ingestionapi/v1/pse/bootstrap" | bash

                                . /tmp/ir_envs

                                cleanup() {
                                    echo "================================"
                                    echo "Job 1: InvisiRisk PSE cleanup"
                                    echo "================================"

                                    pse-data-collector cleanup || true
                                }

                                trap cleanup EXIT

                                node --version
                                npm --version

                                if [ -f package-lock.json ]; then
                                    npm ci --legacy-peer-deps
                                elif [ -f package.json ]; then
                                    npm install --legacy-peer-deps
                                else
                                    echo "package.json not found"
                                    exit 1
                                fi
                            '''
                        }
                    }
                }

                stage('Job 2 - Dependency Check') {
                    steps {
                        dir('job-dependency-check') {
                            deleteDir()
                            checkout scm

                            sh '''
                                set -e

                                echo "================================"
                                echo "Job 2: InvisiRisk PSE setup"
                                echo "================================"

                                curl -sSf \
                                  -H "x-api-key: $IR_TOKEN" \
                                  "$IR_URL/ingestionapi/v1/pse/bootstrap" | bash

                                . /tmp/ir_envs

                                cleanup() {
                                    echo "================================"
                                    echo "Job 2: InvisiRisk PSE cleanup"
                                    echo "================================"

                                    pse-data-collector cleanup || true
                                }

                                trap cleanup EXIT

                                if [ -f package-lock.json ]; then
                                    npm ci --legacy-peer-deps
                                elif [ -f package.json ]; then
                                    npm install --legacy-peer-deps
                                else
                                    echo "package.json not found"
                                    exit 1
                                fi

                                npm ls || true
                            '''
                        }
                    }
                }

                stage('Job 3 - Run Tests') {
                    steps {
                        dir('job-tests') {
                            deleteDir()
                            checkout scm

                            sh '''
                                set -e

                                echo "================================"
                                echo "Job 3: InvisiRisk PSE setup"
                                echo "================================"

                                curl -sSf \
                                  -H "x-api-key: $IR_TOKEN" \
                                  "$IR_URL/ingestionapi/v1/pse/bootstrap" | bash

                                . /tmp/ir_envs

                                cleanup() {
                                    echo "================================"
                                    echo "Job 3: InvisiRisk PSE cleanup"
                                    echo "================================"

                                    pse-data-collector cleanup || true
                                }

                                trap cleanup EXIT

                                if [ -f package-lock.json ]; then
                                    npm ci --legacy-peer-deps
                                elif [ -f package.json ]; then
                                    npm install --legacy-peer-deps
                                else
                                    echo "package.json not found"
                                    exit 1
                                fi

                                npm test
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'All Jenkins jobs completed successfully.'
        }

        failure {
            echo 'One or more Jenkins jobs failed.'
        }

        always {
            echo 'Jenkins pipeline finished.'
        }
    }
}
