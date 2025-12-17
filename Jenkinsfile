pipeline {
    agent any

    tools {
        maven 'maven-3.9.11'
    }

    environment {
        SONAR_HOST_URL = 'http://13.235.62.16:9000'
    }

    stages {

        stage('Checkout') {
            steps {
                cleanWs()
                git branch: 'feature3',
                    url: 'https://github.com/techcoms/springboot-app'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=spring-boot-demo \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=\${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }
         stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: '''
                   --scan .
                   --format HTML
                   --format XML
                  --out target
               ''',
              odcInstallation: 'OWASP-Dependency-Check'
            }
              post {
                always {
                     dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                     archiveArtifacts artifacts: 'target/dependency-check-report.html', fingerprint: true
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
                    docker rm -f mysonarqubeapp || true
                    docker run -d --name mysonarqubeapp -p 81:8080 firstsonarproject:latest
                '''
            }
        }
    }
}
