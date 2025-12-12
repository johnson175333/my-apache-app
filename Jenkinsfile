pipeline {
  agent any

  environment {
    IMAGE = "my-apache-app"
    CN    = "my-apache-app"
  }

  stages {
    stage('Checkout') {
      steps {
        // use your SSH credential id
        git url: 'git@github.com:johnson175333/my-apache-app.git',
            branch: 'main',
            credentialsId: 'github-ssh'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
          set -euo pipefail
          echo "Building image ${IMAGE}:${BUILD_NUMBER}"
          docker build -t ${IMAGE}:latest -t ${IMAGE}:${BUILD_NUMBER} .
        '''
      }
    }

    stage('Run Apache Container') {
      steps {
        sh '''
          set -euo pipefail
          CN=${CN}
          IMG=${IMAGE}:${BUILD_NUMBER}

          echo "Ensuring no previous container named $CN exists"
          if docker ps -a --format '{{.Names}}' | grep -qw "$CN"; then
            docker rm -f "$CN" || true
          fi

          # If port 80 on host is in use, fallback to 8080
          if ss -ltn '( sport = :80 )' | grep -q LISTEN; then
            echo "Port 80 is in use on the host — starting container on host port 8080"
            docker run --name "$CN" -d -p 8080:80 "$IMG"
            echo "Container started at host port 8080"
          else
            echo "Port 80 is free — starting container on host port 80"
            docker run --name "$CN" -d -p 80:80 "$IMG"
            echo "Container started at host port 80"
          fi
        '''
      }
    }
  }

  post {
    always {
      sh 'echo "Docker images on host:"; docker images --format "{{.Repository}}:{{.Tag}} {{.ID}} {{.Size}}" || true'
    }
    success {
      echo "Pipeline finished SUCCESS"
    }
    failure {
      echo "Pipeline finished FAILURE"
    }
  }
}
