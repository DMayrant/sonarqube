pipeline {
    agent any 

    stages {
        stage ('Checkout Code') {
            steps {
                checkout scm 
            }
        }
        stage ('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh '''
                         docker run --rm \
                        -e SONAR_HOST_URL=http://host.docker.internal:9000 \
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
}