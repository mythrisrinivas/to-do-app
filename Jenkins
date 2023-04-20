pipeline {  
    agent any  
    parameters {
        booleanParam(defaultValue: true, description: '', name: 'Build')
    }
    stages {
        stage ('Build') {  
            when { expression { return params.Build }} 
            steps {
                sh "mvn site"
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker build -t ${user}/helloapp:${currentBuild.number} ."
                    sh "docker tag ${user}/helloapp:${currentBuild.number} ${user}/helloapp:latest"
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/site/**'
                }
            }
        }
        stage ('Push to registry') {
            when { expression { return params.Build }} 
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker login -u ${user} -p ${pass}"
                    sh "docker push ${user}/helloapp:${currentBuild.number}"
                    sh "docker push ${user}/helloapp:latest"
                }
            }
        }
        stage ('Deploy') {  
            steps {
                sh "docker stop helloapp || true && docker rm helloapp || true"
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker run -d -p 8085:8085  --name helloapp ${user}/helloapp:latest"
                }
            }
        }
    }  
} 
