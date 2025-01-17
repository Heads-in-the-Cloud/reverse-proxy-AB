#!groovy
pipeline {
    agent any

    environment {
        image_label = "reverse-proxy-ab"
        commit = sh(returnStdout: true, script: "git rev-parse --short=8 HEAD").trim()
        image = ""
    }

    stages {
        stage('Build') {
            steps {
                script {
                    image = docker.build(image_label)
                }
            }
        }

        stage('Push to registry') {
            steps {
                script {
                    docker.withRegistry(
                      ORG_ACCOUNT_NUM
                        + ".dkr.ecr."
                        + region
                        + ".amazonaws.com/reverse-proxy-ab",
                      "ecr:$region:ecr-creds"
                    ) {
                        image.push("$commit")
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

