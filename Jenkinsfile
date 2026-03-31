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
//         stage('SonarQube Analysis') {
//             steps {
//                 script {
//                     def scannerHome = tool 'sonar-scanner'
//                     withSonarQubeEnv('SonarQube') {
//                         withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
//                             sh """
//                             ${scannerHome}/bin/sonar-scanner \
//                             -Dsonar.projectKey=portfolio-website \
//                             -Dsonar.sources=. \
//                             -Dsonar.host.url=http://34.100.253.85:9000 \
//                             -Dsonar.login=$SONAR_TOKEN
//                             """
//                         }
//                     }
//                 }
//             }
//         }
//         stage('Code Quality Check') {
//             steps {
//                 sh '''
//                 htmlhint index.html || true
//                 '''
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
//         stage('Docker Security Scan') {
//             steps {
//                 sh '''
//                 trivy image $IMAGE_NAME:$IMAGE_TAG
//                 '''
//             }
//         }
//         stage('Test Container') {
//             steps {
//                 sh '''
//                 docker run -d -p 8085:80 --name test-container $IMAGE_NAME:$IMAGE_TAG
//                 sleep 10
//                 curl http://localhost:8085
//                 docker rm -f test-container
//                 '''
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
//             stage('Update GitOps Repo') {
//                 steps {
//                     withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
//                         sh """
//                         rm -rf gitops-repo
//                         git clone https://$GIT_USER:$GIT_PASS@github.com/anubharanidharanm-source/Gitops-argoCD-repo.git gitops-repo

//                         cd gitops-repo

//                         sed -i "s|image: .*|image: $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG|g" portfolio-web-app.yaml

//                         git config user.email "jenkins@bharani.tech"
//                         git config user.name "jenkins"

//                         git add .
//                         git commit -m "Update image to version $IMAGE_TAG"
//                         git push origin main
//                         """
//                     }
//                 }
//             }
//     }
// }


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

        stage('Prepare Reports') {
            steps {
                sh 'mkdir -p reports'
            }
        }

        stage('Code Quality Check') {
            steps {
                sh '''
                htmlhint index.html > reports/html-report.txt || true
                '''
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

        stage('Docker Security Scan') {
            steps {
                sh '''
                trivy image --format json -o reports/trivy-report.json $IMAGE_NAME:$IMAGE_TAG
                trivy image --severity HIGH,CRITICAL $IMAGE_NAME:$IMAGE_TAG > reports/trivy-summary.txt || true
                '''
            }
        }

        stage('Test Container') {
            steps {
                sh '''
                docker run -d -p 8085:80 --name test-container $IMAGE_NAME:$IMAGE_TAG
                sleep 10
                curl -I http://localhost:8085 > reports/container-test.txt || true
                docker rm -f test-container
                '''
            }
        }

        stage('Generate DevSecOps Report') {
            steps {
                sh '''
                echo "===============================" > reports/devsecops-report.txt
                echo " DEVSECOPS PIPELINE REPORT" >> reports/devsecops-report.txt
                echo "===============================" >> reports/devsecops-report.txt
                echo "" >> reports/devsecops-report.txt
                echo "BUILD NUMBER: $BUILD_NUMBER" >> reports/devsecops-report.txt
                echo "DATE: $(date)" >> reports/devsecops-report.txt

                echo "" >> reports/devsecops-report.txt
                echo "----- CODE QUALITY CHECK (HTMLHint) -----" >> reports/devsecops-report.txt
                cat reports/html-report.txt >> reports/devsecops-report.txt

                echo "" >> reports/devsecops-report.txt
                echo "----- DOCKER SECURITY SCAN (Trivy) -----" >> reports/devsecops-report.txt
                cat reports/trivy-summary.txt >> reports/devsecops-report.txt

                echo "" >> reports/devsecops-report.txt
                echo "----- CONTAINER TEST -----" >> reports/devsecops-report.txt
                cat reports/container-test.txt >> reports/devsecops-report.txt
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'reports/*', fingerprint: true
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