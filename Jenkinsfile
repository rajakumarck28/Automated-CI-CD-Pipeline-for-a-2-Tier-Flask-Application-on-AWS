pipeline {

    agent any

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/rajakumarck28/Automated-CI-CD-Pipeline-for-a-2-Tier-Flask-Application-on-AWS.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-auth-app .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh 'docker rm -f flask-auth-app || true'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p 5000:5000 --name flask-auth-app flask-auth-app'
            }
        }

    }
}
