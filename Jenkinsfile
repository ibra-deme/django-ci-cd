pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://localhost:9000' // Mettez à jour si nécessaire
        SONARQUBE_CREDENTIALS = credentials('token-sonar')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/ibra-deme/django-ci-cd.git',
                        credentialsId: 'token-git'
                    ]]
                )
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            docker run --rm \
                                -e SONAR_HOST_URL="${SONARQUBE_URL}" \
                                -e SONAR_LOGIN="${SONARQUBE_CREDENTIALS}" \
                                -v "$(pwd):/usr/src" \
                                sonarsource/sonar-scanner-cli:latest
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline failed due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }
    }
}