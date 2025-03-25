pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://host.docker.internal:9000' // Mettez à jour si nécessaire
        SONARQUBE_CREDENTIALS = credentials('my-token')
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
                    userRemoteConfigs: [[
                        credentialsId: 'token-git', 
                        url: 'https://github.com/ibra-deme/django-ci-cd.git'
                    ]]
                )
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh '''
                            docker --version
                            pwd
                            docker run --rm \
                                -e SONAR_HOST_URL="${SONARQUBE_URL}" \
                                -e SONAR_LOGIN="${my-token}" \
                                -v "$(pwd):/usr/src" \
                                -v /var/run/docker.sock:/var/run/docker.sock \
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

        stage('Trivy Scan') {
            steps {
                script {
                    sh '''
                        docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image ${DOCKER_IMAGE}:${DOCKER_TAG}
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
