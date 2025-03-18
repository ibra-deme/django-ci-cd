pipeline {
    agent any

    environment {

        SONARQUBE_URL = 'http://sonarqube:9000' // Mettez à jour si nécessaire
        SONARQUBE_CREDENTIALS = credentials('sonar-token')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], gitTool: 'Default', userRemoteConfigs: [[credentialsId: 'token-git', url: 'https://github.com/ibra-deme/django-ci-cd.git']])
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh '''
                            docker run --rm -e SONAR_HOST_URL="http://172.18.0.3:9000" -e SONAR_LOGIN=****** -v /var/jenkins_home/workspace/django-pipeline:/usr/src sonarsource/sonar-scanner-cli:latest

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
