pipeline {
    agent any

    environment {
        DOCKER_ID = "dockeridd36"  
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh '''
                    docker rm -f test-container || true
                    docker run -d -p 8080:80 --name test-container $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    sh '''
                    curl -f http://localhost:8080
                    '''
                }
            }
        }

        stage('Docker Login & Push') {
            environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS')
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to Dev') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh '''
                    mkdir -p .kube
                    cat $KUBECONFIG > .kube/config

                    cp fastapi/values.yaml values.yaml
                    sed -i "s/tag:.*/tag: ${DOCKER_TAG}/" values.yaml

                    kubectl create namespace dev || true

                    helm upgrade --install app fastapi \
                      --values values.yaml \
                      --namespace dev
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh '''
                    mkdir -p .kube
                    cat $KUBECONFIG > .kube/config

                    cp fastapi/values.yaml values.yaml
                    sed -i "s/tag:.*/tag: ${DOCKER_TAG}/" values.yaml

                    kubectl create namespace staging || true

                    helm upgrade --install app fastapi \
                      --values values.yaml \
                      --namespace staging
                    '''
                }
            }
        }

        stage('Deploy to Production') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Yes'
                }

                script {
                    sh '''
                    mkdir -p .kube
                    cat $KUBECONFIG > .kube/config

                    cp fastapi/values.yaml values.yaml
                    sed -i "s/tag:.*/tag: ${DOCKER_TAG}/" values.yaml

                    kubectl create namespace prod || true

                    helm upgrade --install app fastapi \
                      --values values.yaml \
                      --namespace prod
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            mail to: "yasine.chabnoune@gmail.com",
                 subject: "Jenkins Pipeline FAILED: ${env.JOB_NAME}",
                 body: "Check Jenkins logs: ${env.BUILD_URL}"
        }
    }
}
