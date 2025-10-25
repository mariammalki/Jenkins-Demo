pipeline {
    agent any

    environment {
        registry = "wahidh007/demo-jenkins"
        registryCredential = 'docker-hub-credentials'
        dockerImage = ''
    }

    triggers {
        pollSCM('*/5 * * * *')
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/wahid007/Jenkins-Demo.git'
            }
        }
        
        stage('Build App') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage('Build Docker image') {
            steps {
                script {
                    dockerImage = docker.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker image') {
            steps {
                script {
                    try {
                        docker.withRegistry('', registryCredential) {
                            dockerImage.push()
                        }
                    } catch (err) {
                        echo "Push failed: ${err}"
                        currentBuild.result = 'FAILURE'
                        error("Stopping pipeline due to push failure.")
                    }
                }
            }
        }

        stage('Deploy Docker container') {
            steps {
                script {
                    try {
                        sh "docker stop demo-jenkins || true"
                        sh "docker rm demo-jenkins || true"
                        sh "docker run --name demo-jenkins -d -p 2222:2222 ${registry}:${BUILD_NUMBER}"
                        slackSend(
                            color: "good",
                            message: "${registry}:${BUILD_NUMBER} - container successfully deployed! :man_dancing:"
                        )
                    } catch (err) {
                        slackSend(
                            color: "danger",
                            message: "Deployment failed: ${err} :ghost:"
                        )
                        error("Deployment failed.")
                    }
                }
            }
        }

        // Déploiement K8S et vérification peuvent être ajoutés ici si besoin
    }
    
    post {
        always {
            script {
                echo "Pipeline finished with status: ${currentBuild.currentResult}"
            }
        }
        success {
            script {
                slackSend(
                    color: "good",
                    message: "Pipeline execution successful! :man_dancing:"
                )
            }
        }
        failure {
            script {
                slackSend(
                    color: "danger",
                    message: "Pipeline execution failed! :ghost:"
                )
            }
        }
    }
}
