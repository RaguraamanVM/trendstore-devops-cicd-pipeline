pipeline {

    agent any

    environment {

        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "trend-eks"

        DOCKER_IMAGE = "raguraaman/trendstore"

        IMAGE_TAG = "${BUILD_NUMBER}"

    }

    stages {

        stage('Checkout Source') {

            steps {

                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/RaguraamanVM/trendstore-devops-cicd-pipeline.git'

            }

        }

        stage('Build Docker Image') {

            steps {

                dir('app') {

                    sh """
                    docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                    """

                }

            }

        }

        stage('DockerHub Login') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''

                }

            }

        }

        stage('Push Docker Image') {

            steps {

                sh """

                docker push ${DOCKER_IMAGE}:${IMAGE_TAG}

                docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest

                docker push ${DOCKER_IMAGE}:latest

                """

            }

        }

        stage('Configure AWS') {

            steps {

                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {

                    sh '''

                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID

                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY

                    aws configure set default.region ap-south-1

                    aws eks update-kubeconfig \
                    --region ap-south-1 \
                    --name trend-eks

                    '''

                }

            }

        }

        stage('Deploy to Kubernetes') {

            steps {

                sh """

                sed -i 's|IMAGE_NAME|${DOCKER_IMAGE}:${IMAGE_TAG}|g' k8s/deployment.yaml

                kubectl apply -f k8s/deployment.yaml

                kubectl apply -f k8s/service.yaml

                """

            }

        }

        stage('Verify Deployment') {

            steps {

                sh '''

                kubectl rollout status deployment/trendstore-deployment

                kubectl get pods

                kubectl get svc

                '''

            }

        }

    }

    post {

        success {

            echo "Deployment Successful"

        }

        failure {

            echo "Pipeline Failed"

        }

        always {

            cleanWs()

        }

    }

}
