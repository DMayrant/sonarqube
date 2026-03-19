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
                -v /boot:/boot:ro \
                aquasec/kube-bench:latest \
                --json > kube-bench.json || true
                '''
            }
        }
        stage ('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh '''
                        set -e 

                        echo "SONAR HOST: $SONAR_HOST_URL"
                       
                        docker run --rm \
                         -e SONAR_HOST_URL=$SONAR_HOST_URL \
                         -e SONAR_LOGIN=$SONAR_AUTH_TOKEN \
                         -v $(pwd):/usr/src \
                         sonarsource/sonar-scanner-cli \
                         sonar-scanner \
                            -Dsonar.projectKey=my-app \
                            -Dsonar.sources=.
                        '''
                    }
                }
            }
        }
        stage ('Quality Gate Coverage'){
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
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