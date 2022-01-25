#!groovy
pipeline {
    agent any

    environment {
        COMMIT_HASH = sh(returnStdout: true, script: "git rev-parse --short=8 HEAD").trim()
        AWS_REGION = sh(script:'aws configure get region', returnStdout: true).trim()
        AWS_ACCOUNT_ID = sh(script:'aws sts get-caller-identity --query "Account" --output text', returnStdout: true).trim()
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        PROJECT_ID  = "AB"

        image_label = "reverse-proxy-${PROJECT_ID.toLowerCase()}"
        image = null
        packaged = false
        built = false
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh 'docker context use default'
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
                    docker.withRegistry(
                        "http://$ECR_URI/$image_label",
                        "ecr:$AWS_REGION:jenkins"
                    ) {
                        image.push("$COMMIT_HASH")
                        image.push('latest')
                    }
                }
            }
        }
    }

    post {
        cleanup {
            script {
                if(packaged) {
                    if(built) {
                        sh "docker rmi $image_label"
                    }
                }
            }
        }
    }
}

