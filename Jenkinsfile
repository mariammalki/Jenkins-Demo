def dockerImage

pipeline {
  agent any

  options {
    timestamps()
  }

  environment {
    REGISTRY = "mariem507/demo-jenkins"
    REGISTRY_CRED = "docker-hub-credentials"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  triggers {
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Clone Repo') {
      steps {
        git branch: 'main', url: 'https://github.com/wahid007/Jenkins-Demo.git'
      }
    }

    stage('Build App') {
      steps {
        sh "mvn -v"
        sh "mvn clean package -DskipTests"
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
          try {
            docker.withRegistry('https://index.docker.io/v1/', REGISTRY_CRED) {
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
            sh """
              docker rm -f demo-jenkins || true
              docker run -d --name demo-jenkins -p 2222:2222 ${REGISTRY}:${IMAGE_TAG}
            """
            try {
              slackSend(
                tokenCredentialId: 'slack-token',
                channel: '#builds',
                color: "good",
                message: "${REGISTRY}:${IMAGE_TAG} - container successfully deployed! :man_dancing:"
              )
            } catch (e) {
              echo "Slack non configuré/indisponible: ${e}"
            }
          } catch (err) {
            try {
              slackSend(
                tokenCredentialId: 'slack-token',
                channel: '#builds',
                color: "danger",
                message: "Deployment failed: ${err} :ghost:"
              )
            } catch (e) {
              echo "Slack non configuré/indisponible: ${e}"
            }
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
        try {
          slackSend(
            tokenCredentialId: 'slack-token',
            channel: '#builds',
            color: "good",
            message: "Pipeline execution successful! :man_dancing:"
          )
        } catch (e) {
          echo "Slack non configuré/indisponible: ${e}"
        }
      }
    }
    failure {
      script {
        try {
          slackSend(
            tokenCredentialId: 'slack-token',
            channel: '#builds',
            color: "danger",
            message: "Pipeline execution failed! :ghost:"
          )
        } catch (e) {
          echo "Slack non configuré/indisponible: ${e}"
        }
      }
    }
  }
}
