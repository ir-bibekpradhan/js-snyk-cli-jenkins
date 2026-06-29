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
                sh '''
                    set -e

                    echo "Downloading InvisiRisk PSE bootstrap"

                    curl -sSf -H "x-api-key: ${IR_TOKEN}" \
                      "${IR_URL}/ingestionapi/v1/pse/bootstrap?mode=native&runner=any" \
                      -o /tmp/bootstrap.sh

                    chmod +x /tmp/bootstrap.sh
                    bash /tmp/bootstrap.sh

                    test -f /tmp/ir_envs || {
                      echo "/tmp/ir_envs missing"
                      tail -n 200 /tmp/bootstrap.log || true
                      exit 1
                    }
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    set -e

                    . /tmp/ir_envs

                    node --version || true
                    npm --version || true

                    npm install --legacy-peer-deps
                '''
            }
        }

        stage('Dependency Check') {
            steps {
                sh '''
                    set -e

                    . /tmp/ir_envs

                    npm ls || true
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    . /tmp/ir_envs

                    docker --version || true

                    if [ -f Dockerfile ]; then
                      docker build -t js-snyk-cli-jenkins:latest .
                    else
                      echo "Dockerfile not found, skipping Docker build"
                    fi
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                sh '''
                    set -e

                    mkdir -p jenkins-artifacts

                    zip -r jenkins-artifacts/archive.zip . \
                      -x ".git/*" \
                      -x "node_modules/*" \
                      -x "jenkins-artifacts/*"
                '''

                archiveArtifacts artifacts: 'jenkins-artifacts/archive.zip', fingerprint: true
            }
        }
    }

    post {
        always {
            sh '''
                if command -v pse-data-collector >/dev/null 2>&1; then
                  pse-data-collector cleanup || true
                else
                  echo "pse-data-collector not found, skipping cleanup"
                fi
            '''
        }
    }
}
