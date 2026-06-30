pipeline {
    agent {
        docker {
            image 'python:3.11'
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
                    curl -sSf -H "x-api-key: ${IR_TOKEN}" \
                      "${IR_URL}/ingestionapi/v1/pse/bootstrap" | bash

                    . /tmp/ir_envs

                    pip install -r requirements.txt
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
