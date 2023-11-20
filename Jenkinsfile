pipeline {
    agent {
        kubernetes {
            label 'custom-agent'
            containerTemplate {
                name 'kaniko'
                image 'gcr.io/kaniko-project/executor:latest'
                command 'cat'
                ttyEnabled true
            }
        }
    }

    environment {
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
                    sh "aws sts assume-role --role-arn $AWS_ROLE_TO_ASSUME --role-session-name JenkinsSession --output json > assumed-role.json"
                    sh "aws configure set aws_access_key_id \$(jq -r .Credentials.AccessKeyId assumed-role.json)"
                    sh "aws configure set aws_secret_access_key \$(jq -r .Credentials.SecretAccessKey assumed-role.json)"
                    sh "aws configure set aws_session_token \$(jq -r .Credentials.SessionToken assumed-role.json)"

                    // Move to the vote directory
                    dir('vote') {
                        // Build Docker image using Kaniko
                        sh 'cat Dockerfile | /kaniko/executor --context . --dockerfile Dockerfile --destination $ECR_REGISTRY/vote:latest'

                        // Push the Docker image to ECR
                        sh "docker tag vote $ECR_REGISTRY/vote:latest"
                        sh "docker push $ECR_REGISTRY/vote:latest"
                    }
                }
            }
        }

        stage('Build and Push Worker Image') {
            steps {
                script {
                    // Assume the IAM role
                    sh "aws sts assume-role --role-arn $AWS_ROLE_TO_ASSUME --role-session-name JenkinsSession --output json > assumed-role.json"
                    sh "aws configure set aws_access_key_id \$(jq -r .Credentials.AccessKeyId assumed-role.json)"
                    sh "aws configure set aws_secret_access_key \$(jq -r .Credentials.SecretAccessKey assumed-role.json)"
                    sh "aws configure set aws_session_token \$(jq -r .Credentials.SessionToken assumed-role.json)"

                    // Move to the worker directory
                    dir('worker') {
                        // Build Docker image using Kaniko
                        sh 'cat Dockerfile | /kaniko/executor --context . --dockerfile Dockerfile --destination $ECR_REGISTRY/worker:latest'

                        // Push the Docker image to ECR
                        sh "docker tag worker $ECR_REGISTRY/worker:latest"
                        sh "docker push $ECR_REGISTRY/worker:latest"
                    }
                }
            }
        }

        stage('Build and Push Result Image') {
            steps {
                script {
                    // Assume the IAM role
                    sh "aws sts assume-role --role-arn $AWS_ROLE_TO_ASSUME --role-session-name JenkinsSession --output json > assumed-role.json"
                    sh "aws configure set aws_access_key_id \$(jq -r .Credentials.AccessKeyId assumed-role.json)"
                    sh "aws configure set aws_secret_access_key \$(jq -r .Credentials.SecretAccessKey assumed-role.json)"
                    sh "aws configure set aws_session_token \$(jq -r .Credentials.SessionToken assumed-role.json)"

                    // Move to the result directory
                    dir('result') {
                        // Build Docker image using Kaniko
                        sh 'cat Dockerfile | /kaniko/executor --context . --dockerfile Dockerfile --destination $ECR_REGISTRY/result:latest'

                        // Push the Docker image to ECR
                        sh "docker tag result $ECR_REGISTRY/result:latest"
                        sh "docker push $ECR_REGISTRY/result:latest"
                    }
                }
            }
        }

    }
}

