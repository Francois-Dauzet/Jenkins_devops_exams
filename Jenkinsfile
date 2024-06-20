pipeline {
    environment {
        DOCKER_ID = "francoisdauzet" // replace this with your docker-id
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }
    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Checkout') { // Check out the code from GitHub
            steps {
                git url: 'https://github.com/Francois-Dauzet/Jenkins_devops_exams.git', branch: 'master'
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
        stage('Push Docker Images') { // Push the Docker images to Docker Hub
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // retrieve docker password from secret text saved in Jenkins
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
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace dev
                    '''
                }
            }
        }
        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                // Create an Approval Button with a timeout of 15minutes.
                // This requires a manual validation in order to deploy on production environment
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }
}
