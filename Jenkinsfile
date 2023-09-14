pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
        KUBECTL_HOST_IP = "54.81.162.182"

    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t amitdevops12/node-app:${DOCKER_TAG}"
            }
        }
        stage('Docker Hub Push'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'github_creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker push amitdevops12/node-app:${DOCKER_TAG}"
                }
            }
        }
        stage('Docker Deploy K8s'){
            steps{
                sh "pwd"
	    	    sh "chmod +x changeTag.sh"
		        sh "./changeTag.sh ${DOCKER_TAG}"

                sshagent(['minikube-server']) {
                    withCredentials([sshUserPrivateKey(credentialsId: 'public-all1', keyFileVariable: 'SSH_KEY_MINIKUBE_SERVER')]) {
                    
                        sh "scp -o StrictHostKeyChecking=no service.yaml node-app-pod.yaml ec2-user@34.229.192.91:/home/ec2-user"
		                sh "ssh ec2-user@${KUBECTL_HOST_IP} docker login -u amitdevops12 -p ${docker-hubPwd}"
		                sh "ssh ec2-user@${KUBECTL_HOST_IP} kubectl apply -f /home/ec2-user/node-app-pod.yaml"
		                sh "ssh ec2-user@${KUBECTL_HOST_IP} kubectl apply -f /home/ec2-user/service.yaml"

				
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
