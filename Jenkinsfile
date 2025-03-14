pipeline {
    agent any
    parameters {
        string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'car-image', description: 'Name of the Docker image')
    }
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws_credential')
    }

    stages {
        // Git Checkout Stage
        stage('Code checkout from Git') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']], 
                    extensions: [], 
                    userRemoteConfigs: [[credentialsId: 'jenkin-git', url: 'https://github.com/remyars18/jenkins-aws-test.git']]
                )
            }
        }

        // Docker Build Stage
        stage('Build Docker Image') {
            steps {
                script {
                    // Log in to Docker Hub before building the image
                    withCredentials([usernamePassword(credentialsId: 'jenkins-dockerhub', passwordVariable: 'pwd', usernameVariable: 'usr')]) {
                        sh "echo '${pwd}' | docker login -u ${usr} --password-stdin"
                    }
                    
                    // Build the Docker image
                    sh "docker build -t ${params.DOCKER_IMAGE_NAME}:latest -f Dockerfile ."
                }
            }
        }

        // Docker Push Stage
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub using credentials
                    withCredentials([usernamePassword(credentialsId: 'jenkins-dockerhub', passwordVariable: 'pwd', usernameVariable: 'user')]) {
                        sh "echo '${pwd}' | docker login -u ${user} --password-stdin"
                        
                        // Tag and push the built Docker image to Docker Hub
                        sh "docker tag ${params.DOCKER_IMAGE_NAME}:latest ${user}/${params.DOCKER_IMAGE_NAME}:latest"
                        sh "docker push ${user}/${params.DOCKER_IMAGE_NAME}:latest"
                    }
                }
            }
        }

        // EKS Connection Test Stage
        stage('EKS Connection Test') {
            steps {
                script {
                    // Ensure the withKubeConfig step is correctly set
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kube-config-eks-file', namespace: '', restrictKubeConfigAccess: false, serverUrl: ''){
                        sh 'kubectl get nodes'
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                        sh 'kubectl get svc car-app-svc -o wide'
                    }
                }
            }
        }
    }
}
