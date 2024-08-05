pipeline {
    agent any
    environment {
        registry = "654654178716.dkr.ecr.us-east-1.amazonaws.com/ecs-jenkins"
        region = "us-east-1"
        clusterName = "ecs-jenkins-cluster" // Your ECS cluster name
        serviceName = "ecs-jenkins-service-new" // Your ECS service name
        containerName = "ecs-jenkins-container" // Your container name in the ECS task definition
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/uday912/Ecs-project-3.git'
            }
        }

        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws_cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh 'aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}'
                        sh 'docker push ${registry}'
                    }
                }
            }
        }

        stage('Stop previous containers') {
            steps {
                sh 'docker ps -f name=myContainer -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -f name=myContainer -q | xargs -r docker container rm'
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    sh 'docker run -d -p 80:80 --rm --name myContainer ${registry}:latest'
                }
            }
        }

        stage('Update ECS Task Definition') {
            steps {
                script {
                    // Fetch the current task definition
                    def taskDefArn = sh(
                        script: "aws ecs describe-services --cluster ${clusterName} --services ${serviceName} --region ${region} | jq -r '.services[0].taskDefinition'",
                        returnStdout: true
                    ).trim()

                    // Create a new revision of the task definition with the updated image
                    sh """
                    aws ecs register-task-definition \
                        --family ecs-jenkins-task-family \
                        --container-definitions '[{
                            "name": "${containerName}",
                            "image": "${registry}:latest",
                            "essential": true,
                            "portMappings": [{
                                "containerPort": 80,
                                "hostPort": 80
                            }],
                            "memory": 512
                        }]' \
                        --region ${region}
                    """
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    // Get the new task definition revision
                    def newTaskDefArn = sh(
                        script: "aws ecs describe-task-definition --task-definition ecs-jenkins-task-family --region ${region} | jq -r '.taskDefinition.taskDefinitionArn'",
                        returnStdout: true
                    ).trim()

                    // Update ECS service to use the new task definition
                    sh "aws ecs update-service --cluster ${clusterName} --service ${serviceName} --task-definition ${newTaskDefArn} --region ${region}"
                }
            }
        }
    }
}
