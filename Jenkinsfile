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
                // Using the specialized DSL for the Dependency Check Plugin
                dependencyCheck additionalArguments: """
                    --scan .
                    --format HTML
                    --format XML
                    --out target
                    --nvdApiKey ${NVD_API_KEY}
                """, odcInstallation: 'DC'
                
                dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/dependency-check-report.html, target/dependency-check-report.xml',
                                     allowEmptyArchive: true
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
                // Added a stop/rm command to prevent "Port already in use" errors on repeat runs
                sh """
                    docker stop spring-app || true
                    docker rm spring-app || true
                    docker run -d --name spring-app -p 8181:8080 ${DOCKERHUB_REPO}:${BUILD_NUMBER}
                """
            }
        }
    }
}
