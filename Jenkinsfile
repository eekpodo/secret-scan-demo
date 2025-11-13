pipeline {
    agent any

    environment {
        DETECT_VENV = "${WORKSPACE}/.scan-venv"
        TRUFFLE_VENV = "${WORKSPACE}/.scan-venv-truffle"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/eekpodo/secret-scan-demo.git', branch: 'main'
            }
        }

        stage('Secrets Scan') {
            parallel {

                stage('DetectSecrets') {
                    steps {
                        sh """#!/bin/bash
set -e
python3 -m venv ${DETECT_VENV}
source ${DETECT_VENV}/bin/activate
pip install --upgrade pip
pip install detect-secrets
detect-secrets scan --all-files > .secrets.baseline
SECRET_COUNT=\$(detect-secrets audit .secrets.baseline --non-interactive | grep -c 'Potential secrets' || true)
if [ "\$SECRET_COUNT" -gt 0 ]; then
    echo "DetectSecrets found secrets. Failing build."
    exit 1
fi
"""
                    }
                }

                stage('TruffleHog') {
                    steps {
                        sh """#!/bin/bash
set -e
python3 -m venv ${TRUFFLE_VENV}
source ${TRUFFLE_VENV}/bin/activate
pip install --upgrade pip
pip install trufflehog3 jq
trufflehog3 --no-history --json . --output truffle-report.json
SECRET_COUNT=\$(jq '. | length' truffle-report.json)
if [ "\$SECRET_COUNT" -gt 0 ]; then
    echo "TruffleHog found secrets. Failing build."
    exit 1
fi
"""
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '.secrets.baseline, truffle-report.json', allowEmptyArchive: true
            echo "Secrets scan finished."
        }
        failure {
            echo "Build failed due to detected secrets!"
        }
        success {
            echo "No secrets detected. Build passed."
        }
    }
}
