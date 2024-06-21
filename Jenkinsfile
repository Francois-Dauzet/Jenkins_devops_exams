pipeline {
    environment {
        DOCKER_ID = "francoisdauzet"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Francois-Dauzet/Jenkins_devops_exams.git', branch: 'master'
            }
        }
        stage('Build Docker Images') {
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
        stage('Run Docker Containers') {
            parallel {
                stage('Run Movie Service') {
                    steps {
                        script {
                            sh '''
                            docker rm -f movie-service || true
                            docker run -d -p 8001:8000 --name movie-service $DOCKER_ID/movie-service:$DOCKER_TAG
                            '''
                        }
                    }
                }
                stage('Run Cast Service') {
                    steps {
                        script {
                            sh '''
                            docker rm -f cast-service || true
                            docker run -d -p 8002:8000 --name cast-service $DOCKER_ID/cast-service:$DOCKER_TAG
                            '''
                        }
                    }
                }
            }
        }
        stage('Push Docker Images') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
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
        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    echo "$KUBECONFIG" > .kube/config

                    # Deploy movie-service
                    cp movie-service/helm/values.yaml movie_values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" movie_values.yml
                    helm upgrade --install app movie-service/helm --values=movie_values.yml --namespace dev

                    # Deploy cast-service
                    cp cast-service/helm/values.yaml cast_values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" cast_values.yml
                    helm upgrade --install app cast-service/helm --values=cast_values.yml --namespace dev
                    '''
                }
            }
        }
        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    echo "$KUBECONFIG" > .kube/config

                    # Deploy movie-service
                    cp movie-service/helm/values.yaml movie_values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" movie_values.yml
                    helm upgrade --install app movie-service/helm --values=movie_values.yml --namespace staging

                    # Deploy cast-service
                    cp cast-service/helm/values.yaml cast_values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" cast_values.yml
                    helm upgrade --install app cast-service/helm --values=cast_values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    echo "$KUBECONFIG" > .kube/config

                    # Deploy movie-service
                    cp movie-service/helm/values.yaml movie_values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" movie_values.yml
                    helm upgrade --install app movie-service/helm --values=movie_values.yml --namespace prod

                    # Deploy cast-service
                    cp cast-service/helm/values.yaml cast_values.yml
                    sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" cast_values.yml
                    helm upgrade --install app cast-service/helm --values=cast_values.yml --namespace prod
                    '''
                }
            }
        }
    }
}
