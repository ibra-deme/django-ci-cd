pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://localhost:9000'
        sonar= "sonar"
        SONARQUBE_CREDENTIALS = credentials('token-sonar')
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
                            sonar-scanner
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