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
    }
}
