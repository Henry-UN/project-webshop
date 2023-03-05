pipeline {
    agent any

    stages {
        stage('Gitclone') {
            steps {
                sh 'git clone https://github.com/Henry-UN/project-webshop.git'
            }
        }    
        stage('Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerpass', usernameVariable: 'dockeruser')]) {
                    sh 'cd project-webshop/app/cartservice && docker build -t cartservice .'
                    sh 'docker login -u $dockeruser -p $dockerpass'
                    sh 'docker tag cartservice $dockeruser/cartservice:latest'
                    sh 'docker push $dockeruser/cartservice:latest'
                    // Remove all images 
                    sh 'docker rmi -f $(docker images -aq) && docker system prune -f'
                    // Remove git hub repo
                    sh 'cd ../../../'
                    sh 'rm -rf project-webshop'
                }
            }
        }
        stage("connect to deploy server") {
            environment { 
                SSH_CRED = credentials('k8s-server')
            }
            steps {
                script {
                    sh """
                    #!/bin/bash
                    ssh -i "$SSH_CRED" -o StrictHostKeyChecking=no ubuntu@ec2-18-222-116-38.us-east-2.compute.amazonaws.com << EOF
                    git clone https://github.com/Henry-UN/project-webshop.git
                    cd project-webshop
                    git checkout adservice
                    kubectl apply -f adservice.yaml
                    exit
                    EOF
                    """
                }
            }
        }			
    }
}
