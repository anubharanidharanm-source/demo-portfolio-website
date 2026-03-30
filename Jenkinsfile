// pipeline {
//     agent { label 'GCP-Jenkins-Agent' }

//     stages {

//         stage('Clone Repo') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 sh 'docker build -t portfolio-app .'
//             }
//         }

//         stage('Run Container') {
//             steps {
//                 sh 'docker run -d -p 8090:80 --name portfolio-container portfolio-app'
//             }
//         }

//     }
// }

pipeline {
    agent { label 'GCP-Jenkins-Agent' }

    environment {
        IMAGE_NAME = "anubharani/portfolio-website"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f portfolio-container || true
                docker run -d -p 8091:80 $IMAGE_NAME:latest
                '''
            }
        }

    }
}