pipeline {
    agent { label 'GCP-Jenkins-Agent' }

    environment {
        IMAGE_NAME = "anubharani/portfolio-website"
        IMAGE_TAG = "${BUILD_NUMBER}"
        GCP_ARTIFACT_IMAGE_NAME = "asia-south1-docker.pkg.dev/testing-bharani/portfolio-artifact-registry/portfolio-website"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQube') {
                        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                            sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=portfolio-website \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://34.100.253.85:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                docker tag $IMAGE_NAME:$IMAGE_TAG $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG
                docker tag $IMAGE_NAME:$IMAGE_TAG $GCP_ARTIFACT_IMAGE_NAME:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker push $IMAGE_NAME:latest
                    """
                }
            }
        }

        stage('Push to GCP Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GCP_KEY')]) {
                    sh """
                    gcloud auth activate-service-account --key-file=$GCP_KEY
                    gcloud auth configure-docker asia-south1-docker.pkg.dev
                    docker push $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG
                    docker push $GCP_ARTIFACT_IMAGE_NAME:latest
                    """
                }
            }
        }

        stage('Run Container') {
            steps {
                sh """
                docker rm -f portfolio-container || true
                docker run -d -p 8091:80 --name portfolio-container $IMAGE_NAME:$IMAGE_TAG
                """
            }
        }
            stage('Update GitOps Repo') {
                steps {
                    withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh """
                        rm -rf gitops-repo
                        git clone https://$GIT_USER:$GIT_PASS@github.com/anubharanidharanm-source/Gitops-argoCD-repo.git gitops-repo

                        cd gitops-repo

                        sed -i "s|image: .*|image: $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG|g" portfolio-web-app.yaml

                        git config user.email "jenkins@bharani.tech"
                        git config user.name "jenkins"

                        git add .
                        git commit -m "Update image to version $IMAGE_TAG"
                        git push origin main
                        """
                    }
                }
            }
    }
}


