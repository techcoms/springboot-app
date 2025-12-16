pipeline {
    agent any

    tools {
        maven 'maven-3.9.11'
    }

    environment {
        SONAR_HOST_URL = 'http://13.235.62.16:9000/'
        SONAR_AUTH_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/techcoms/springboot-app', branch: 'feature3'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh '''
                    mvn sonar:sonar \
                      -Dsonar.projectKey=spring-boot-demo \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                '''
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                sh 'mvn clean verify'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/dependency-check-report.*', fingerprint: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t firstsonarproject:latest .'
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker run -d --name mysonarqubeapp -p 81:8080 firstsonarproject:latest
                '''
            }
        }
    }
}
