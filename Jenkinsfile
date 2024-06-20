pipeline {
    environment {
        DOCKER_ID = "francoisdauzet" // replace this with your docker-id
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
        DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins
        KUBECONFIG = credentials("KUBECONFIG") // we retrieve kubeconfig from secret file called config saved on jenkins
        GITHUB_CREDENTIALS = credentials("GITHUB_CREDENTIALS_ID") // replace with the ID of your GitHub credentials
    }
    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Checkout') { // Check out the code from GitHub
            steps {
                git url: 'https://github.com/Francois-Dauzet/Jenkins_devops_exams.git', branch: 'master', credentialsId: "${GITHUB_CREDENTIALS}"
            }
        }
        stage('Build Docker Images') { // Build Docker images for the services
            parallel {
                stage('Build Movie Service') {
                    steps {
                        script {
                            sh '''
                            cd movie-service
                            docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG .
                            '''
                        }
                    }
                }
                stage('Build Cast Service') {
                    steps {
                        script {
                            sh '''
                            cd cast-service
                            docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG .
                            '''
                        }
                    }
                }
            }
        }
        stage('Run Docker Containers') { // Run the containers to ensure they work
            parallel {
                stage('Run Movie Service') {
                    steps {
                        script {
                            sh '''
                            docker run -d -p 8001:8000 --name movie-service $DOCKER_ID/movie-service:$DOCKER_TAG
                            sleep 10
                            curl localhost:8001
                            docker rm -f movie-service
                            '''
                        }
                    }
                }
                stage('Run Cast Service') {
                    steps {
                        script {
                            sh '''
                            docker run -d -p 8002:8000 --name cast-service $DOCKER_ID/cast-service:$DOCKER_TAG
                            sleep 10
                            curl localhost:8002
                            docker rm -f cast-service
                            '''
                        }
                    }
                }
            }
        }
        stage('Push Docker Images') { // Push the Docker images to Docker Hub
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/movie-service:$DOCKER_TAG
                    docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') { // Deploy to Kubernetes environments
            parallel {
                stage('Deploy to Dev') {
                    steps {
                        script {
                            sh '''
                            mkdir -p ~/.kube
                            echo "$KUBECONFIG" > ~/.kube/config
                            kubectl config set-context --current --namespace=dev
                            helm upgrade --install movie-service movie-service/helm --set image.tag=$DOCKER_TAG --namespace dev
                            helm upgrade --install cast-service cast-service/helm --set image.tag=$DOCKER_TAG --namespace dev
                            '''
                        }
                    }
                }
                stage('Deploy to Staging') {
                    steps {
                        script {
                            sh '''
                            kubectl config set-context --current --namespace=staging
                            helm upgrade --install movie-service movie-service/helm --set image.tag=$DOCKER_TAG --namespace staging
                            helm upgrade --install cast-service cast-service/helm --set image.tag=$DOCKER_TAG --namespace staging
                            '''
                        }
                    }
                }
                stage('Deploy to Prod') {
                    steps {
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to deploy in production?', ok: 'Yes'
                        }
                        script {
                            sh '''
                            kubectl config set-context --current --namespace=prod
                            helm upgrade --install movie-service movie-service/helm --set image.tag=$DOCKER_TAG --namespace prod
                            helm upgrade --install cast-service cast-service/helm --set image.tag=$DOCKER_TAG --namespace prod
                            '''
                        }
                    }
                }
            }
        }
    }
}
