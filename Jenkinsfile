pipeline {
    agent { label 'GCP-Jenkins-Agent' }

    environment {
        IMAGE_NAME = "anubharani/portfolio-website"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
            }
        }
        stage('Lint or Validate') {
            agent {
                docker { image 'node:20-alpine' }
            }
            steps {
                sh '''
                npm install -g htmlhint stylelint eslint

                echo "Running HTMLHint..."
                htmlhint **/*.html

                echo "Running Stylelint..."
                stylelint **/*.css

                echo "Running ESLint..."
                eslint **/*.js || true
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f portfolio-container || true
                docker run -d -p 8091:80 --name portfolio-container $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

    }
}