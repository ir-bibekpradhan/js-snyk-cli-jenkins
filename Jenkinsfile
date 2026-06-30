pipeline {
    agent {
        docker {
            image 'python:3.11'
            args '-u root'
        }
    }

    environment {
        IR_URL = 'https://app.stage.invisirisk.com'
        IR_TOKEN = credentials('IR_API_KEY')
    }

    stages {
        stage('Install Requirements') {
            steps {
                sh '''
                    set -e

                    # InvisiRisk BAF/PSE setup
                    curl -sSf -H "x-api-key: ${IR_TOKEN}" \
                      "${IR_URL}/ingestionapi/v1/pse/bootstrap" | bash

                    . /tmp/ir_envs

                    # Existing user scripts
                    if [ -f requirements.txt ]; then
                      pip install -r requirements.txt
                    else
                      echo "requirements.txt not found, skipping pip install"
                    fi
                '''
            }
        }
    }

    post {
        always {
            sh '''
                pse-data-collector cleanup || true
            '''
        }
    }
}
