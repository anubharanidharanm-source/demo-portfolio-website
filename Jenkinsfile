pipeline {
    agent { label 'gcp-agent' }

    stages {

        stage('Clone Repo') {
            steps {
                git 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t portfolio-app .'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p 8081:80 portfolio-app'
            }
        }

    }
}