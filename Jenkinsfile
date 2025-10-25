def dockerImage

pipeline {
    agent any

    options {
        timestamps()
        // Timeout global pour éviter que le push Docker bloque indéfiniment
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        REGISTRY = "mariem507/demo-jenkins"
        REGISTRY_CRED = "docker-hub-credentials" // Vérifier que ce credential existe dans Jenkins
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        SLACK_CRED = "slack-token" // Vérifier que ce credential existe dans Jenkins
        SLACK_CHANNEL = "#builds"
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/mariammalki/Jenkins-Demo.git'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn -v'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    sh 'docker version'
                    dockerImage = docker.build("${REGISTRY}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', REGISTRY_CRED) {
                        // Retry push en cas de problème réseau
                        retry(3) {
                            dockerImage.push()
                        }
                    }
                }
            }
        }

        stage('Deploy Docker container') {
            steps {
                script {
                    try {
                        sh """
                            docker rm -f demo-jenkins || true
                            docker run -d --name demo-jenkins -p 2222:2222 ${REGISTRY}:${IMAGE_TAG}
                        """
                        slackSend(
                            tokenCredentialId: SLACK_CRED,
                            channel: SLACK_CHANNEL,
                            color: "good",
                            message: "${REGISTRY}:${IMAGE_TAG} - container successfully deployed! :man_dancing:"
                        )
                    } catch (err) {
                        slackSend(
                            tokenCredentialId: SLACK_CRED,
                            channel: SLACK_CHANNEL,
                            color: "danger",
                            message: "Deployment failed: ${err} :ghost:"
                        )
                        error("Deployment failed.")
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
        success {
            script {
                slackSend(
                    tokenCredentialId: SLACK_CRED,
                    channel: SLACK_CHANNEL,
                    color: "good",
                    message: "Pipeline execution successful! :man_dancing:"
                )
            }
        }
        failure {
            script {
                slackSend(
                    tokenCredentialId: SLACK_CRED,
                    channel: SLACK_CHANNEL,
                    color: "danger",
                    message: "Pipeline execution failed! :ghost:"
                )
            }
        }
    }
}
