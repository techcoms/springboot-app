pipeline{
     options {
    buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '6'))
  }
    agent any
     environment {
        DOCKERHUB_REPO = "techcoms/backend-springboot-maven-jar"
     }
    tools{
        maven "maven3.9.1"
    }
     stages{
        stage("git checkout"){
           steps{
                  git branch: "main", url: "https://github.com/techcoms/springboot-app.git"
            }
        } 
        stage("build artifacts with maven"){
            steps{
                sh "mvn clean package"
            }
        }
        stage('build docker image'){
            steps{
                sh "docker build -t ${DOCKERHUB_REPO}:${BUILD_NUMBER} ."
            }
        }
         stage('run a docker container'){
                steps{
                   sh "docker run -d -p 8181:8080 ${DOCKERHUB_REPO}:${BUILD_NUMBER}"
                }
         }
        stage("login to dockerhub and push image"){
            steps { 
                withCredentials([usernamePassword(credentialsId: 'docker-creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) { 
                    sh "docker login -u $USERNAME -p $PASSWORD "
                    sh "docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}"
                  }
              }
         }  
     }
}   
