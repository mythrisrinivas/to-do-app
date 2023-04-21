pipeline {  
    agent any
    environment {
        ACCESS_KEY = credentials('TO_DO_ACCESS_KEY')
        SECRET_KEY = credentials('TO_DO_SECRET_KEY')

    }
    stages {
        stage ('Build') {  
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker build -t ${user}/infra:${currentBuild.number} ."
                    sh "docker tag ${user}/infra:${currentBuild.number} ${user}/helloapp:latest"
                }
            }
        }
        stage ('Push to registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker login -u ${user} -p ${pass}"
                    sh "docker push ${user}/infra:${currentBuild.number}"
                    sh "docker push ${user}/infra:latest"
                }
            }
        }
        stage ('Docker Deploy') {
            steps {
                sh "docker stop helloapp || true && docker rm helloapp || true"
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker run -d -p 8085:8085  --name helloapp ${user}/helloapp:latest"
                }
            }
        }
        stage ('Deploy on EKS') {
             steps {
                    sh "cd terraform"
                    sh "terraform init"
                    sh "terraform plan"
                    sh "terraform apply -var aws_secret_key=${ACCESS_KEY} -var aws_secret_key=${SECRET_KEY} --auto-approve"
             }
        }
    }  
} 
