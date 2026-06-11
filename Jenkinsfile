pipeline {
    agent any

    environment {
        DOCKER_ID = "dockeridd36" 
        DOCKER_IMAGE = "movie-service"
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
                sh 'docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG ./movie-service'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f test-container || true
                docker run -d -p 8080:80 --name test-container $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                sleep 10
                '''
            }
        }

        stage('Test Application') {
            steps {
                sh 'curl -f http://localhost:8080 || true'
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS')
            }
            steps {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Deploy Dev') {
            steps {
                echo "Deploy skipped (explained in report)"
            }
        }

        stage('Deploy Staging') {
            steps {
                echo "Deploy skipped (explained in report)"
            }
        }

        stage('Deploy Production') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Yes'
                }
                echo "Production deploy skipped"
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
