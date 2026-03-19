pipeline {
    agent any 

    stages {
        stage ('Checkout Code') {
            steps {
                checkout scm 
            }
        }
        stage ('Kubescape Scan') {
            steps {
                sh '''
                set -euo pipefail 

                echo 'Scanning Kubernetes cluster...'
                kubescape scan framework nsa,mitre \
                  --format json \
                  --output kubescape-results.json || true
                '''
            }
        }
        stage ('Kube-bench Scan') {
            steps {
                sh '''
                set -euo pipefail 

                echo 'Running Kube-bench...'
                docker run --rm \
                --pid=host \
                -v /etc:/etc:ro \
                -v /var:/var:ro \
                -v /usr:/usr:ro \
                aquasec/kube-bench:latest --json > kube-bench.json || true
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '*.json', allowEmptyArchive: true
        }
        success {
            echo "Pipeline executed successful ✅"
        }
        failure {
            echo "Pipeline failed, please check logs 🚫"
        } 
    }
}