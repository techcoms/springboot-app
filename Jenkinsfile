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
                sh 'mvn clean test'
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
            environment {
                NVD_API_KEY = credentials('nvd-api-key')
            }
            steps {
                sh """
                    mvn org.owasp:dependency-check-maven:9.0.9:check \
                      -Dnvd.api.key=\${NVD_API_KEY} \
                      -Dnvd.api.delay=6000 \
                      -Dnvd.api.maxRetryCount=15 \
                      -DautoUpdate=false \
                      -DfailOnError=false
                """
            }
            post {
                always {
                    script {
                        if (fileExists('target/dependency-check-report.html')) {
                            archiveArtifacts artifacts: 'target/dependency-check-report.html',
                                             fingerprint: true
                        } else {
                            echo 'Dependency-Check report not generated'
                        }
                    }
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
