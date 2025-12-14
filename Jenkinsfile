pipeline {
  agent any

  environment {
    IMAGE = "my-apache-app"
    CN    = "my-apache-app"
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'git@github.com:johnson175333/my-apache-app.git',
            branch: 'main',
            credentialsId: 'github-pat'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh """
          bash -lc '
            set -euo pipefail
            echo "Building image ${IMAGE}:${BUILD_NUMBER}"
            docker build -t ${IMAGE}:latest -t ${IMAGE}:${BUILD_NUMBER} .
          '
        """
      }
    }

    stage('Run Apache Container') {
      steps {
        sh """
          bash -lc '
            set -euo pipefail
            IMG="${IMAGE}:${BUILD_NUMBER}"
            CN="${CN}"

            echo "Removing any existing container named $CN"
            if docker ps -a --format "{{.Names}}" | grep -qw "$CN"; then
              docker rm -f "$CN" || true
            fi

            if ss -ltn "( sport = :80 )" | grep -q LISTEN; then
              echo "Host port 80 in use — using 8080"
              docker run --name "$CN" -d -p 8080:80 "$IMG"
              echo "Started $CN → host:8080"
            else
              echo "Host port 80 free — using 80"
              docker run --name "$CN" -d -p 80:80 "$IMG"
              echo "Started $CN → host:80"
            fi
          '
        """
      }
    }
  }

  post {
    always {
      sh "bash -lc 'echo Docker images on host; docker images --format \"{{.Repository}}:{{.Tag}} {{.ID}} {{.Size}}\" || true'"
    }
    success {
      echo "Pipeline finished SUCCESS"
    }
    failure {
      echo "Pipeline finished FAILURE"
    }
  }
}
