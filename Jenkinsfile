pipeline {
    agent any

    environment {
       // SONARQUBE_URL = 'http://sonarqube:9000' // Mettez à jour si nécessaire
       // SONARQUBE_CREDENTIALS = credentials('sonar-token')
       // PATH = "/opt/sonar-scanner/bin:${env.PATH}"
        DOCKER_IMAGE = 'ibrademe/django-ci-cd-app'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    gitTool: 'Default',
                    userRemoteConfigs: [[credentialsId: 'token-git', url: 'https://github.com/ibra-deme/django-ci-cd.git']]
                )
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    // Scanner avec Trivy via Docker
                    sh '''
                        docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image ibrademe/django-ci-cd-app:latest
                    '''
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub') {
                        sh '''
                            docker --version
                            docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} -f Dockerfile .
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
            }
        }
    }
}
