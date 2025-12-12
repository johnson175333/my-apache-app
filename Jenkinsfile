pipeline {
    agent any

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/johnson175333/apache-jenkins-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t apache-jenkins .'
            }
        }

        stage('Run Apache Container') {
            steps {
                sh '''
                docker stop apache || true
                docker rm apache || true
                docker run -d -p 80:80 --name apache apache-jenkins
                '''
            }
        }
    }
}
