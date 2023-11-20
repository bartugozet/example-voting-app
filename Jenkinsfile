pipeline {
    agent any
    environment {
        // Define your ECR registry URL
        ECR_REGISTRY = '130575395405.dkr.ecr.us-east-1.amazonaws.com'
        AWS_ROLE_TO_ASSUME = 'arn:aws:iam::130575395405:role/talent_role'
        AWS_REGION = 'us-east-1'
        AWS_CREDENTIALS_ID = 'bartu-ecr'
    }

    stages {
        stage('Build and Push Vote Image') {
            steps {
                script {
                    // Assume the IAM role
                    withAWS(region: AWS_REGION, credentials: AWS_CREDENTIALS_ID, roleAccount: 'account-id') {
                        // Move to the vote directory
                        dir('vote') {
                            // Build Docker image
                            sh 'docker build -t vote .'

                            // Tag the Docker image for ECR
                            docker.withRegistry('', "ecr:${AWS_REGION}") {
                                sh "docker tag vote $ECR_REGISTRY/vote:latest"
                            }

                            // Push the Docker image to ECR
                            docker.withRegistry('', "ecr:${AWS_REGION}") {
                                sh "docker push $ECR_REGISTRY/vote:latest"
                            }
                        }
                    }
                }
            }
        }
        stage('Build and Push Worker Image') {
            steps {
                script {
                    // Assume the IAM role
                    withAWS(region: AWS_REGION, credentials: AWS_CREDENTIALS_ID, roleAccount: 'account-id') {
                        // Move to the worker directory
                        dir('worker') {
                            sh 'docker build -t worker .'
                            docker.withRegistry('', "ecr:${AWS_REGION}") {
                                sh "docker tag worker $ECR_REGISTRY/worker:latest"
                            }
                            docker.withRegistry('', "ecr:${AWS_REGION}") {
                                sh "docker push $ECR_REGISTRY/worker:latest"
                            }
                        }
                    }
                }
            }
        }
        stage('Build and Push Result Image') {
            steps {
                script {
                    // Assume the IAM role
                    withAWS(region: AWS_REGION, credentials: AWS_CREDENTIALS_ID, roleAccount: 'account-id') {
                        // Move to the result directory
                        dir('result') {
                            sh 'docker build -t result .'
                            docker.withRegistry('', "ecr:${AWS_REGION}") {
                                sh "docker tag result $ECR_REGISTRY/result:latest"
                            }
                            docker.withRegistry('', "ecr:${AWS_REGION}") {
                                sh "docker push $ECR_REGISTRY/result:latest"
                            }
                        }
                    }
                }
	    }
        }
    }
}
