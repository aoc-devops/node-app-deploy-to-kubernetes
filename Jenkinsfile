pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()

    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t amitdevops12/node-app:${DOCKER_TAG}"
            }
        }
        stage('Docker Hub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'docker-hubPwd')]) {
                    sh "docker login -u amitdevops12 -p ${docker-hubPwd}"
                    sh "docker push amitdevops12/node-app:${DOCKER_TAG}"
                }
            }
        }
        stage('Docker Deploy K8s'){
            steps{

                sshagent(['tomcat-dev']) {
                    withCredentials([string(credentialsId: 'nexus-pwd', variable: 'nexusPwd')]) {
                        sh "ssh ec2-user@172.31.0.38 docker login -u admin -p ${nexusPwd} ${NEXUS_URL}"
                    }
					// Remove existing container, if container name does not exists still proceed with the build
					sh script: "ssh ec2-user@172.31.0.38 docker rm -f nodeapp",  returnStatus: true
                    
                    sh "ssh ec2-user@172.31.0.38 docker run -d -p 8080:8080 --name nodeapp ${IMAGE_URL_WITH_TAG}"
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
