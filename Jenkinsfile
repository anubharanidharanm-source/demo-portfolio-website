pipeline {
    agent { label 'GCP-Jenkins-Agent' }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t portfolio-app .'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p 8090:80 --name portfolio-container portfolio-app'
            }
        }

    }
}