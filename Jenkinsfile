pipeline {
    agent any
     environment {
        registry = "654654178716.dkr.ecr.us-east-1.amazonaws.com/ecs-jenkins"
    }
   
    stages {
          stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/uday912/Ecs-project-3.git'
            }
        }
           stage('Building image') {
             steps{
                  script {
                   dockerImage = docker.build registry
                   }
      }
           }
    
            stage('Pushing to ECR') {
             steps{  
                  script {
               withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws_cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 654654178716.dkr.ecr.us-east-1.amazonaws.com'
     sh 'docker push 654654178716.dkr.ecr.us-east-1.amazonaws.com/ecs-jenkins'
}

}
                  }
            }
             stage('stop previous containers') {
               steps {
            sh 'docker ps -f name=myContainer -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=myContainer -q | xargs -r docker container rm'
         }
       }
            stage('Docker Run') {
              steps{
                   script {
                sh 'docker run -d -p 80:80 --rm --name myContainer 654654178716.dkr.ecr.us-east-1.amazonaws.com/ecs-jenkins:latest'     
      }
    }
        }
    }
  }
