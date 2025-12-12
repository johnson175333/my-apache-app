pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git url: 'git@github.com:johnson175333/my-apache-app.git',
            branch: 'main',               // confirm main or master
            credentialsId: 'github-ssh'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t my-apache-app .'
      }
    }

    stage('Run Apache Container') {
      steps {
        sh 'docker run -d -p 80:80 my-apache-app'
      }
    }
  }
}
