pipeline {
    agent any
    parameters {
        string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'car-image', description: 'Name of the Docker image')
    }
    
    stages {
        stage('Code checkout from Git') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']], 
                    extensions: [], 
                    userRemoteConfigs: [[credentialsId: 'jenkin-git', url: 'https://github.com/remyars18/jenkins-aws-test.git']]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "sudo docker build -t ${params.DOCKER_IMAGE_NAME}:latest -f Dockerfile ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Assuming 'jenkin-dockerhub' is the ID of your Docker Hub credentials
                    withCredentials([usernamePassword(credentialsId: 'jenkin-dockerhub', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        // Log in to Docker Hub using provided credentials
                        sh "echo '${DOCKER_HUB_PASSWORD}' | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        
                        // Tag and push the built Docker image to Docker Hub
                        sh "docker tag ${params.DOCKER_IMAGE_NAME}:latest ${DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
                        sh "docker push ${DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
                    }
                }
            }
        }
    }
}
