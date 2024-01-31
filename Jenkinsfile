pipeline {
    agent none
    environment {
        IMAGE_NAME = "elinka/common-numbers"
        HUB_CRED_ID = "elinka_docker_hub"
        PROJECT_DIR = "common-numbers"
    }
    stages {
        stage("build") {
            agent any
            steps {
                sh "docker build -t ${IMAGE_NAME}:${GIT_COMMIT} -t ${IMAGE_NAME}:latest ."
            }
        }
     stage("sonar scan") {
            agent any
            steps {
                withCredentials(
                    [
                    string(credentialsId: "sonarqube_url", variable: "SONARQUBE_URL"),
                    usernamePassword(credentialsId: "elinka_sonar_token2", usernameVariable: "PROJECT_KEY",
                    passwordVariable: "PROJECT_TOKEN")
                    ]
                )
                {
                    sh  '''docker run \
                        --rm \
                        -e SONAR_HOST_URL="${SONARQUBE_URL}" \
                        -e SONAR_SCANNER_OPTS="-Dsonar.projectKey=${PROJECT_KEY}" \
                        -e SONAR_TOKEN="${PROJECT_TOKEN}" \
                        -v "${WORKSPACE}:/usr/src" \
                        sonarsource/sonar-scanner-cli'''
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
        stage("deploy") {
            agent any
            steps {
                withCredentials(
                    [
                        string(credentialsId: "production_ip", variable: 'SERVER_IP'),
                        sshUserPrivateKey(credentialsId: "production_key", keyFileVariable: 'SERVER_KEY', usernameVariable: 'SERVER_USERNAME')
                    ]
                ) {
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} mkdir -p ${PROJECT_DIR}'
                    sh 'scp -i ${SERVER_KEY} docker-compose.yaml ${SERVER_USERNAME}@${SERVER_IP}:${PROJECT_DIR}'
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} docker compose -f ${PROJECT_DIR}/docker-compose.yaml pull'
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} docker compose -f ${PROJECT_DIR}/docker-compose.yaml up -d'
                }
            }
        }
    }
}