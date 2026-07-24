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
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Job 1 - Install Dependencies') {
            steps {
                sh '''
                    set -e

                    echo "================================"
                    echo "Job 1: InvisiRisk PSE setup"
                    echo "================================"

                    sudo mkdir -p /pse-bin
                    sudo chown -R jenkins:jenkins /pse-bin

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

        stage('Job 2 - Dependency Check') {
            steps {
                sh '''
                    set -e

                    echo "================================"
                    echo "Job 2: InvisiRisk PSE setup"
                    echo "================================"

                    sudo mkdir -p /pse-bin
                    sudo chown -R jenkins:jenkins /pse-bin

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

        stage('Job 3 - Run Tests') {
            steps {
                sh '''
                    set -e

                    echo "================================"
                    echo "Job 3: InvisiRisk PSE setup"
                    echo "================================"

                    sudo mkdir -p /pse-bin
                    sudo chown -R jenkins:jenkins /pse-bin

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

    post {
        success {
            echo 'All Jenkins stages completed successfully.'
        }

        failure {
            echo 'One or more Jenkins stages failed.'
        }

        always {
            echo 'Jenkins pipeline finished.'
        }
    }
}
