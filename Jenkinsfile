pipeline { 
    agent any
    environment {
        DOCKERHUB_REPO = "techcoms/backend-springboot-maven-war"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "13.235.62.16:8081"
        NEXUS_REPOSITORY = "maven-snapshots"
        NEXUS_CREDENTIAL_ID = "nexusrepo"
        SONAR_URL = "http://your-sonarqube-ip:9000" // Update this
        SONAR_TOKEN_ID = "sonarqube-token"       // Credential ID in Jenkins
    }
    tools {
        maven "maven-3.9.11"
    }
    stages {
        stage("Git Checkout") {
            steps {
                git branch: 'Feature', credentialsId: 'github-creds', 
                    url: 'https://github.com/techcoms/backend-springboot-maven-war.git'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') { // Match the name in Jenkins Global Tools
                        sh "mvn sonar:sonar -Dsonar.projectKey=backend-app -Dsonar.host.url=${SONAR_URL}"
                    }
                }
            }
        }

        stage("Trivy FS Scan") {
            steps {
                // Scans the source code for secrets and vulnerabilities before building
                sh "trivy fs . > trivy-fs-report.txt"
            }
        }

        stage("Build Artifact") {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    if(filesByGlob.length > 0) {
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION, protocol: NEXUS_PROTOCOL, nexusUrl: NEXUS_URL,
                            groupId: pom.groupId, version: pom.version, repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [[artifactId: pom.artifactId, classifier: '', file: filesByGlob[0].path, type: pom.packaging]]
                        )
                    }
                }
            }
        }

        stage("Docker Build & Scan") {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}:${BUILD_NUMBER} ."
                // Scans the image for vulnerabilities. exit-code 0 ensures it doesn't fail the build yet.
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKERHUB_REPO}:${BUILD_NUMBER}"
            }
        }

        stage("Push to DockerHub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) { 
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    sh "docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                // Ensure you have the 'kubernetes' plugin and 'kubeconfig' credentials
                withKubeConfig([credentialsId: 'k8s-creds']) {
                    sh "kubectl set image deployment/backend-deployment backend-container=${DOCKERHUB_REPO}:${BUILD_NUMBER} --record"
                    sh "kubectl get pods"
                }
            }
        }
    }
    post {
        always {
            mail to: "jyothiprakashg05@gmail.com",
                 subject: "Build ${currentBuild.fullDisplayName}: ${currentBuild.currentResult}",
                 body: "Project: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}"
        }
    }
}
