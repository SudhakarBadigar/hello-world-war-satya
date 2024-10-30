//add 
pipeline { 
 agent { label "slave-1"}
  //agent any
  stages{ 
    stage('maven compile'){ 
    steps{ 
      script{ 
        sh "mvn compile" 
      }
    }
  } 
    stage('maven validate'){ 
    steps{ 
      script{ 
        sh "mvn validate" 
      }
    }
  } 
    stage('maven test'){ 
    steps{ 
      script{ 
        sh "mvn test" 
      }
    }
  } 
  stage('generte package'){ 
    steps{ 
      script{ 
        sh "mvn install" 
      }
    }
  } 
  stage ("sonar analysis") {
      steps {
        script {
          withSonarQubeEnv('SONARQUBE_SERVER') {
              sh "mvn sonar:sonar -Dsonar.projectKey=java-hello"
          }
        }
      }
    }
    stage ("nexus war file push") {
      steps {
        script { 
          withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASSWORD')]) {
           sh "mvn deploy -s settings.xml"
        } 
        }
      }
    }
    stage ("docker build") {
    steps{ 
      script{ 
        sh "docker build -t sudhakar24/devops:${BUILD_NUMBER} ." 
      }
    }
  } 
    stage('docker image pushing'){ 
    steps{ 
      script{ 
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin" 
                        sh "docker push sudhakar24/devops:${BUILD_NUMBER}"
       }
      }
    }
  } 
stage('Deploy') {
            input {
                message 'Do you want to deploy to production?'
                ok 'Deploy!'
                parameters {
                    string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: 'Deployment environment')
                }
            }
             steps { 
             script{ 
                echo "Deploy"  
                 sh "docker stop satya3"
                sh "docker rm satya3"
                sh "docker run -itd --name satya3 -p 9000:8080 sudhakar24/devops:${BUILD_NUMBER}" 
                
        } 
      } 
    }  
}
post {
        always {
            echo 'Cleaning up workspace...'
           // cleanWs()  // This cleans the workspace at the end of the pipeline
        }
    }
}
