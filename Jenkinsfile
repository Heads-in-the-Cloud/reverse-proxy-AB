#!groovy
pipeline {
    agent any

    environment {
        image_label = "reverse-proxy-ab"
        git_commit_hash = "${sh(returnStdout: true, script: 'git rev-parse --short=8 HEAD')}"
        image = ""
    }

    stages {
        stage('Build') {
            steps {
                script {
                    image = docker.build(image_label, "reverse-proxy")
                }
            }
        }

        stage('Push to registry') {
            steps {
                script {
                    docker.withRegistry(API_GATEWAY_ECR_URI_AB, "ecr:$region:ecr-creds") {
                        //image.push("$git_commit_hash")
                        image.push('latest')
                    }
                }
            }
        }

        stage('Clean up') {
            steps {
                sh "docker rmi $image_label"
            }
        }
    }
}

