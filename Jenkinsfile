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
                sh '''
                docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG ./movie-service
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f jenkins || true
                docker run -d -p 80:80 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                sleep 10
                '''
            }
        }

        stage('Test Acceptance') {
            steps {
                sh '''
                curl localhost || true
                '''
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Deploy Dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                sh '''
                rm -rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config

                cp charts/values.yaml values.yaml
                sed -i "s/tag:.*/tag: ${DOCKER_TAG}/" values.yaml

                kubectl create namespace dev || true

                helm upgrade --install app charts \
                  --values values.yaml \
                  --namespace dev
                '''
            }
        }

        stage('Deploy Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                sh '''
                rm -rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config

                cp charts/values.yaml values.yaml
                sed -i "s/tag:.*/tag: ${DOCKER_TAG}/" values.yaml

                kubectl create namespace staging || true

                helm upgrade --install app charts \
                  --values values.yaml \
                  --namespace staging
                '''
            }
        }

        stage('Deploy Production') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Yes'
                }

                sh '''
                rm -rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config

                cp charts/values.yaml values.yaml
                sed -i "s/tag:.*/tag: ${DOCKER_TAG}/" values.yaml

                kubectl create namespace prod || true

                helm upgrade --install app charts \
                  --values values.yaml \
                  --namespace prod
                '''
            }
        }
    }

    post {
        success {
            mail to: "yasine.chabnoune@gmail.com",
                 subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_ID}",
                 body: "Pipeline OK: ${env.BUILD_URL}"
        }
        failure {
            mail to: "yasine.chabnoune@gmail.com",
                 subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_ID}",
                 body: "Pipeline FAILED: ${env.BUILD_URL}"
        }
    }
}
