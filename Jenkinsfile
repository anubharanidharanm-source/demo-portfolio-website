// pipeline {
//     agent { label 'GCP-Jenkins-Agent' }

//     environment {
//         IMAGE_NAME = "anubharani/portfolio-website"
//         IMAGE_TAG = "${BUILD_NUMBER}"
//         GCP_ARTIFACT_IMAGE_NAME = "asia-south1-docker.pkg.dev/testing-bharani/portfolio-artifact-registry/portfolio-website"
//     }

//     stages {

//         stage('Clone Repo') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 sh """
//                 docker build -t $IMAGE_NAME:$IMAGE_TAG .
//                 docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
//                 docker tag $IMAGE_NAME:$IMAGE_TAG $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG
//                 docker tag $IMAGE_NAME:$IMAGE_TAG $GCP_ARTIFACT_IMAGE_NAME:latest
//                 """
//             }
//         }

//         stage('Push to Docker Hub') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
//                     sh """
//                     echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
//                     docker push $IMAGE_NAME:$IMAGE_TAG
//                     docker push $IMAGE_NAME:latest
//                     """
//                 }
//             }
//         }

//         stage('Push to GCP Artifact Registry') {
//             steps {
//                 withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GCP_KEY')]) {
//                     sh """
//                     gcloud auth activate-service-account --key-file=$GCP_KEY
//                     gcloud auth configure-docker asia-south1-docker.pkg.dev
//                     docker push $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG
//                     docker push $GCP_ARTIFACT_IMAGE_NAME:latest
//                     """
//                 }
//             }
//         }

//         stage('Run Container') {
//             steps {
//                 sh """
//                 docker rm -f portfolio-container || true
//                 docker run -d -p 8091:80 --name portfolio-container $IMAGE_NAME:$IMAGE_TAG
//                 """
//             }
//         }

//     }
// }


pipeline {
    agent { label 'GCP-Jenkins-Agent' }

    environment {
        IMAGE_NAME = "anubharani/portfolio-website"
        IMAGE_TAG = "${BUILD_NUMBER}"
        GCP_ARTIFACT_IMAGE_NAME = "asia-south1-docker.pkg.dev/testing-bharani/portfolio-artifact-registry/portfolio-website"
        SONAR_HOST_URL = "http://localhost:9000"
        SONAR_LOGIN = "sqp_05774885b0860b42024f00e7f8ceb99798190b95"
        SONAR_PROJECT_KEY = "portfolio-website"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/anubharanidharanm-source/demo-portfolio-website.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                docker run --rm \
                  --network host \
                  -v "\$(pwd)":/usr/src \
                  sonarsource/sonar-scanner-cli:latest \
                  -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                  -Dsonar.sources=/usr/src \
                  -Dsonar.host.url=${SONAR_HOST_URL} \
                  -Dsonar.login=${SONAR_LOGIN}
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${GCP_ARTIFACT_IMAGE_NAME}:${IMAGE_TAG}
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${GCP_ARTIFACT_IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Push to GCP Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GCP_KEY')]) {
                    sh """
                    gcloud auth activate-service-account --key-file=${GCP_KEY}
                    gcloud auth configure-docker asia-south1-docker.pkg.dev
                    docker push ${GCP_ARTIFACT_IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${GCP_ARTIFACT_IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Run Container') {
            steps {
                sh """
                docker rm -f portfolio-container || true
                docker run -d -p 8091:80 --name portfolio-container ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

    }
}