pipeline {
    agent {
        docker {
            image 'node:20-bookworm'
            args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
            reuseNode true
        }
    }

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(
            name: 'RUN_DOCKER_ARCHIVE_BUILD',
            defaultValue: true,
            description: 'Also run the Docker image build from .github/workflows/irhaha.yml and archive archive.zip'
        )
    }

    environment {
        IR_URL = 'https://app.stage.invisirisk.com'
        DEBUG = 'true'

        // Jenkins credential: Secret text with ID IR_API_KEY
        IR_TOKEN = credentials('IR_API_KEY')
    }

    stages {
        stage('Install Base Tools') {
            steps {
                sh '''
                    set -e

                    apt-get update
                    apt-get install -y --no-install-recommends \
                      bash \
                      ca-certificates \
                      curl \
                      git \
                      gzip \
                      procps \
                      tar \
                      unzip \
                      zip

                    node --version
                    npm --version
                    git --version
                '''
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

                    test -s /tmp/bootstrap.sh || {
                      echo "bootstrap download failed"
                      exit 1
                    }

                    chmod +x /tmp/bootstrap.sh
                    bash /tmp/bootstrap.sh

                    test -f /tmp/ir_envs || {
                      echo "/tmp/ir_envs missing"
                      tail -n 200 /tmp/bootstrap.log || true
                      exit 1
                    }

                    echo "PSE environment file created at /tmp/ir_envs"
                '''
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install NPM Dependencies') {
            steps {
                sh '''
                    set -e

                    # Source PSE variables in the same shell as the build command.
                    . /tmp/ir_envs

                    npm install --legacy-peer-deps
                '''
            }
        }

        stage('Show Dependency Tree') {
            steps {
                sh '''
                    set -e

                    . /tmp/ir_envs

                    # The GitHub workflow runs `npm ls`. This repository may have intentionally
                    # unusual dependency versions, so do not fail the whole Jenkins build only
                    # because npm reports peer/dependency tree warnings.
                    npm ls || true
                '''
            }
        }

        stage('Docker Build and Extract archive.zip') {
            when {
                expression { return params.RUN_DOCKER_ARCHIVE_BUILD }
            }
            steps {
                sh '''
                    set -e

                    . /tmp/ir_envs

                    if ! command -v docker >/dev/null 2>&1; then
                      apt-get update
                      apt-get install -y --no-install-recommends docker.io
                    fi

                    docker version

                    DOCKER_BUILDKIT=1 docker build --no-cache \
                      -t my-app:latest \
                      --build-arg BUILDKIT_SYNTAX=public.ecr.aws/w3c0c0n7/invisirisk/baf-buildkit:latest \
                      --secret id=pse-ca,src=${PSE_CA_CERT_PATH} \
                      --build-arg PSE_PROXY=http://${PSE_PROXY_IP}:3128 \
                      .

                    container_id=$(docker create my-app:latest)
                    docker cp "$container_id:/app/archive.zip" ./archive.zip
                    docker rm "$container_id"
                    ls -lh archive.zip
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: 'archive.zip', fingerprint: true, onlyIfSuccessful: true
                }
            }
        }
    }

    post {
        always {
            sh '''
                if [ -f /tmp/ir_envs ]; then
                  . /tmp/ir_envs || true
                fi

                pse-data-collector cleanup || true
            '''
        }
    }
}
