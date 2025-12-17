pipeline {
    agent any

    tools {
        maven 'maven-3.9.11'
    }

    environment {
        SONAR_HOST_URL = 'http://13.235.62.16:9000'
        DOCKERHUB_REPO = "techcoms/backend-springboot-maven-jar"
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
                          -Dsonar.login=${SONAR_AUTH_TOKEN}
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
                      -Dnvd.api.key=${NVD_API_KEY} \
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
                sh "docker build -t ${DOCKERHUB_REPO}:${BUILD_NUMBER} ."
            }
        }

        stage('Trivy Image Scan') {
            steps {
                // Scans the local image created in the previous stage
                sh """
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --no-progress \
                      ${DOCKERHUB_REPO}:${BUILD_NUMBER}
                """
            }
        }

        stage('Login and Push to DockerHub') {
            steps { 
                withCredentials([usernamePassword(credentialsId: 'docker-creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) { 
                    sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                    sh "docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                // Use --rm or a specific name to avoid container name conflicts on re-runs
                sh "docker run -d -p 8181:8080 ${DOCKERHUB_REPO}:${BUILD_NUMBER}"
            }
        }
    }
}
