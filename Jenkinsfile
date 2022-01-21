#!groovy
pipeline {
    agent any

    environment {
        image_label = "reverse-proxy-ab"
        commit = sh(returnStdout: true, script: "git rev-parse --short=8 HEAD").trim()
        image = null
        built = false
    }

    stages {
        stage('Build') {
            steps {
                script {
                    image = docker.build(image_label)
                }
            }
            post {
                success {
                    script {
                        built = true
                    }
                }
            }
        }

        stage('Push to registry') {
            steps {
                script {
                    ecr_repo_uri ="https://${ORG_ACCOUNT_NUM}.dkr.ecr.${region}.amazonaws.com/${image_label}"
                    docker.withRegistry(ecr_repo_uri, "ecr:$region:ecr-creds") {
                        image.push("$commit")
                        image.push('latest')
                    }
                }
            }
        }
    }
    post {
        cleanup {
            script {
                if(built) {
                    sh "docker rmi $image_label"
                }
            }
        }
    }
}

