pipeline {
    environment {
        DOCKER_ID = "francoisdauzet"
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
        KUBECONFIG = credentials("KUBECONFIG")
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/votre-utilisateur/votre-depot.git'
            }
        }
        stage('Build and Push Docker Images') {
            parallel {
                stage('Build Movie Service') {
                    steps {
                        script {
                            sh '''
                            cd movie-service
                            docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG .
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/movie-service:$DOCKER_TAG
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
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                            '''
                        }
                    }
                }
            }
        }
        stage('Deploy to Dev') {
            steps {
                script {
                    sh '''
                    mkdir -p ~/.kube
                    echo $KUBECONFIG > ~/.kube/config
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
