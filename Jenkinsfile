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
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy Docker container') {
            steps {
                sh "docker stop demo-jenkins || true && docker rm demo-jenkins || true"
                sh "docker run --name demo-jenkins -d -p 2222:2222 ${registry}:${BUILD_NUMBER}"
                script {
                    slackSend(
                        color: "good",
                        message: "${registry}:${BUILD_NUMBER} - image successfully created! :man_dancing:"
                    )
                }
            }
        }

        // Déploiement K8S (optionnel)
        // stage('Deploy K8S') {
        //     steps {
        //         sh "kubectl apply -f k8s.yml"
        //     }
        // }

        // Vérification du déploiement (optionnel)
        // stage('Verify deployment') {
        //     steps {
        //         sh "kubectl get pods"
        //         sh "kubectl get svc"
        //     }
        // }

        // Nettoyage local des images Docker (optionnel)
        // stage('Cleaning up') {
        //     steps {
        //         sh "docker rmi ${registry}:${BUILD_NUMBER}"
        //     }
        // }
    }
    
    post {
        success {
            echo 'Pipeline execution successful!'
            script {
                slackSend(
                    color: "good",
                    message: "Pipeline execution successful! :man_dancing:"
                )
            }
        }
        failure {
            echo 'Pipeline execution failed.'
            script {
                slackSend(
                    color: "danger",
                    message: "Pipeline execution failed! :ghost:"
                )
            }
        }
    }
}
