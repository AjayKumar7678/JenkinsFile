pipeline {
    agent any

    stages {
        stage("Pull source code from GITHUB"){
            steps {
              git 'https://github.com/AjayKumar7678/mywebapp.git'
            }
        }
        
        stage("Build Docker File"){
            steps {
              sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
              sh 'docker image tag $JOB_NAME:v1.$BUILD_ID ajayarya/$JOB_NAME:v1.$BUILD_ID'
              sh 'docker image tag $JOB_NAME:v1.$BUILD_ID ajayarya/$JOB_NAME:latest'
            }
        }
        
        stage("Push Image to Docker HUB"){
            steps {
              withCredentials([string(credentialsId: 'Docker_Hubpassword', variable: 'Docker_Hubpassword')]) {
    // some block
              sh 'docker login -u ajayarya -p ${Docker_Hubpassword}'
              sh 'docker image push ajayarya/$JOB_NAME:v1.$BUILD_ID'
              sh 'docker image push ajayarya/$JOB_NAME:latest'
              sh 'docker image rmi $JOB_NAME:v1.$BUILD_ID ajayarya/$JOB_NAME:v1.$BUILD_ID ajayarya/$JOB_NAME:latest'
}
            }
        }
        
        stage("Docker container deployement"){
            steps {
              sshagent(['dockerhostpassword']) {
    // some block
              sh "ssh -o StrictHostKeyChecking=no root@34.125.45.85 docker container rm -f myapplication"
              sh "ssh -o StrictHostKeyChecking=no root@34.125.45.85 docker image rmi -f docker.io/ajayarya/webapp"
              sh "ssh -o StrictHostKeyChecking=no root@34.125.45.85 docker run -p 8080:80 -d --name myapplication ajayarya/webapp:latest"
}
            }
        }
    }
}
