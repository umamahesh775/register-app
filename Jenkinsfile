pipeline {
    agent {
        docker {
            image 'maven:3.8.6-openjdk-17'
            args '-v /root/.m2:/root/.m2'
        }
    }

    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "umamahesh445/register-app"
        SONARQUBE_ENV = "sonarqube-server"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/umamahesh775/register-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=register-app -Dsonar.sources=src"
                }
            }
        }

        stage('Wait for SonarQube Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${imageTag} ."
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${imageTag}"
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                sh 'mvn clean'
            }
        }
    }
}
