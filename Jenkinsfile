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
	    	sh "chmod +x changeTag.sh"
		sh "./dockerTag.sh ${DOCKER_TAG}"

                sshagent(['minikube-server']) {
                    
                    sh "scp -o StrictHostKeyChecking=no service.yaml node-app-pod.yaml ec2-user@34.229.192.91:/home/ec2-user"
		    sh "ssh ec2-user@34.229.192.91 docker login -u amitdevops12 -p ${docker-hubPwd}"
		    sh "ssh ec2-user@34.229.192.91 kubectl apply -f /home/ec2-user/node-app-pod.yaml"
		    sh "ssh ec2-user@34.229.192.91 kubectl apply -f /home/ec2-user/service.yaml"

				
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
