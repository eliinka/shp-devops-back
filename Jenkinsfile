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
    stage("push") {
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: "${HUB_CRED_ID}",
                usernameVariable: 'HUB_USERNAME', passwordVariable: 'HUB_PASSWORD')]) {
                    sh 'docker login -u ${HUB_USERNAME} -p ${HUB_PASSWORD}'
                    sh 'docker push ${IMAGE_NAME}:${GIT_COMMIT}'
                    sh 'docker push ${IMAGE_NAME}:latest'
                }
            }
        }
}