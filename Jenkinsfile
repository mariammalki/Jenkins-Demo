// Utiliser une variable Groovy (pas une variable d'environnement) pour stocker l'objet Image
def dockerImage

pipeline {
  agent any

  options {
    timestamps()
  }

  environment {
    // ⚠️ Mets ici TON namespace Docker Hub (pas celui de l'enseignant)
    REGISTRY = "mariammalki/demo-jenkins"        // ex. "mariammalki/demo-jenkins"
    REGISTRY_CRED = "docker-hub-credentials"     // ID du credential Jenkins (Username with password)
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    // DOCKER_HOST = 'tcp://IP_DOCKER:2375'      // <-- Décommente si tu utilises Docker distant (2375)
    // DOCKER_BUILDKIT = '1'                     // Optionnel : activer BuildKit
  }

  triggers {
    // Meilleure pratique que */5 : répartit la charge
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Clone Repo') {
      steps {
        // Utilise ton fork si tu en as un : "https://github.com/mariammalki/Jenkins-Demo.git"
        git branch: 'main', url: 'https://github.com/wahid007/Jenkins-Demo.git'
      }
    }

    stage('Build App') {
      steps {
        sh "mvn -v"
        sh "mvn clean package -DskipTests"  // Accélère tant que tu mets au point
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          sh 'docker version'
          // On construit l'image et on stocke l'objet retourné dans la variable Groovy
          dockerImage = docker.build("${REGISTRY}:${IMAGE_TAG}")
        }
      }
    }

    stage('Push Docker image') {
      steps {
        script {
          try {
            // Pour Docker Hub, l'endpoint v1 est attendu par withRegistry
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
            // ✅ Slack (facultatif) — protégé par try/catch pour ne jamais casser le run
            try {
              slackSend(
                tokenCredentialId: 'slack-token', // <-- ID "Secret text" contenant ton xoxb-... Slack
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
