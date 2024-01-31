pipeline {
    agent none
    environment {
        IMAGE_NAME = "elinka/common-numbers"
    }
    stages {
        stage("build") {
            agent any
            steps {
                sh "docker build -t ${IMAGE_NAME}:${GIT_COMMIT} -t ${IMAGE_NAME}:latest ."
            }
        }
    }
}