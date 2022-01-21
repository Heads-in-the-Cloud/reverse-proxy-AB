#!groovy
pipeline {
    agent any

    environment {
        image_label = "reverse-proxy-ab"
        commit = sh(returnStdout: true, script: "git rev-parse --short=8 HEAD").trim()
        image = ""
        ecr_repo_uri ="${ORG_ACCOUNT_NUM}.dkr.ecr.${region}.amazonaws.com/reverse-proxy-ab" 
    }

    stages {
        stage('Build') {
            steps {
                script {
                    image = docker.build(image_label)
                }
            }
            post {
                cleanup {
                    sh "docker rmi $image_label"
                    sh "docker rmi $ecr_repo_uri:latest"
                    sh "docker rmi $ecr_repo_uri:$commit"
                }
            }
        }

        stage('Push to registry') {
            steps {
                script {
                    docker.withRegistry(ecr_repo_uri, "ecr:$region:ecr-creds") {
                        image.push("$commit")
                        image.push('latest')
                    }
                }
            }
        }
    }
}

