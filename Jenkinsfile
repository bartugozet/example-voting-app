pipeline {
    agent {
        kubernetes {
            // Define your Kubernetes pod template for Kaniko
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    args:
    - --dockerfile=/workspace/worker/Dockerfile
    - --context=/workspace
    - --destination=<your-docker-registry>/<your-repository>/worker:latest
  volumes:
  - name: workspace
    emptyDir: {}
"""
        }
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Checkout the repository
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/bartugozet/example-voting-app.git']]])
            }
        }

        stage('Build Docker Image with Kaniko') {
            steps {
                script {
                    // Execute Kaniko to build the Docker image
                    container('kaniko') {
                        sh "/kaniko/executor --context /workspace --dockerfile /workspace/worker/Dockerfile --destination <your-docker-registry>/<your-repository>/worker:latest"
                    }
                }
            }
        }
    }
}

