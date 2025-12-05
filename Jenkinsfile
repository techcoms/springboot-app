pipeline {
    agent any

    tools {
        maven 'mvn'
    }

    environment {
        SONAR_HOST_URL = 'http://3.27.78.81:9000'
        SONAR_AUTH_TOKEN = credentials('sonarqube')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/AnushaAkkena/POC3.git', branch: 'feature3'
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
