pipeline {
    agent any
    environment {
        registry = "850924742604.dkr.ecr.us-east-2.amazonaws.com/mydockerrepo"
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    buildTag = "${commitHash}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build("${registry}:${buildTag}")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 850924742604.dkr.ecr.us-east-2.amazonaws.com'
                    dockerImage.push("${buildTag}")
                    dockerImage.push("latest")
                }
            }
        }
    }
}
