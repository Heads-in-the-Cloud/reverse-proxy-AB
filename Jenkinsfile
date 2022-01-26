#!groovy
pipeline {
    agent any

    parameters {
        string(
            name: 'ProjectId',
            defaultValue: 'AB-utopia',
            description: 'Identifier applied to all names'
        )
    }

    environment {
        COMMIT_HASH = sh(
            script: "git rev-parse --short=8 HEAD",
            returnStdout: true
        ).trim()
        REGION = sh(
            script:'aws configure get region',
            returnStdout: true
        ).trim()
        AWS_ACCOUNT_ID = sh(
            script:'aws sts get-caller-identity --query "Account" --output text',
            returnStdout: true
        ).trim()

        ecr_uri = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com"
        image_label = "${params.ProjectId.toLowerCase()}-reverse-proxy"
        image = null
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
                        "http://$ecr_uri/$image_label",
                        "ecr:$REGION:jenkins"
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
                if(built) {
                    sh "docker rmi $image_label"
                }
            }
        }
    }
}

