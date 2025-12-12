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
            credentialsId: 'github-ssh'
      }
    }

    stage('Build Docker Image') {
      steps {
        // run under bash so pipefail works
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

            # compute image tag inside shell (avoid groovy variable usage)
            IMG="${IMAGE}:${BUILD_NUMBER}"
            CN="${CN}"

            echo "Removing any existing container named $CN (if present)"
            if docker ps -a --format "{{.Names}}" | grep -qw "$CN"; then
              docker rm -f "$CN" || true
            fi

            # choose port
            if ss -ltn "( sport = :80 )" | grep -q LISTEN; then
              echo "Host port 80 is busy — using 8080"
              docker run --name "$CN" -d -p 8080:80 "$IMG"
              echo "Started $CN on host:8080"
            else
              echo "Host port 80 is free — using 80"
              docker run --name "$CN" -d -p 80:80 "$IMG"
              echo "Started $CN on host:80"
            fi
          '
        """
      }
    }
  }

  post {
    always {
      sh "bash -lc 'echo \"Docker images on host:\"; docker images --format \"{{.Repository}}:{{.Tag}} {{.ID}} {{.Size}}\" || true'"
    }
    success {
      echo "Pipeline finished SUCCESS"
    }
    failure {
      echo "Pipeline finished FAILURE"
    }
  }
}
