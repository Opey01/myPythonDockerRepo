pipeline {
    agent any
    environment {
        registry = "850924742604.dkr.ecr.us-east-2.amazonaws.com/mydockerrepo"
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    // Generate unique tag: commit short SHA + build number
                    commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    buildTag = "${commitHash}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // Build Docker image with unique tag
                    dockerImage = docker.build("${registry}:${buildTag}")
                }
            }
        }

        stage('Push to ECR') {
            steps {  
                script {
                    // Authenticate Docker to ECR
                    sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 850924742604.dkr.ecr.us-east-2.amazonaws.com'
                    
                    // Push only unique tag (no :latest to avoid immutability issue)
                    dockerImage.push("${buildTag}")
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    // Stop and remove old container if running
                    sh 'docker ps -f name=mypythonContainer -q | xargs -r docker container stop'
                    sh 'docker container ls -a -f name=mypythonContainer -q | xargs -r docker container rm'

                    // Run new container with the unique tag
                    sh "docker run -d -p 8096:5000 --rm --name mypythonContainer ${registry}:${buildTag}"
                }
            }
        }
    }
}

