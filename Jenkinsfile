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
        // use bash explicitly so pipefail works
        sh """
          bash -lc '
            set -euo pipefail
            echo "Building ${IMAGE}:${BUILD_NUMBER}"
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
            CN=${CN}
            IMG=${IMAGE}:${BUILD_NUMBER}

            echo "Removing any existing container named $CN"
            if docker ps -a --format "{{.Names}}" | grep -qw "$CN"; then
              docker rm -f "$CN" || true
            fi

            # choose host port: 80 if free, else 8080
            if ss -ltn "( sport = :80 )" | grep -q LISTEN; then
              echo "Host port 80 is in use, falling back to 8080"
              docker run --name "$CN" -d -p 8080:80 "$IMG"
              echo "Started container on host port 8080"
            else
              echo "Host port 80 is free, starting on 80"
              docker run --name "$CN" -d -p 80:80 "$IMG"
              echo "Started container on host port 80"
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
    success { echo "Pipeline finished SUCCESS" }
    failure { echo "Pipeline finished FAILURE" }
  }
}
