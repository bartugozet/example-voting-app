pipeline {
 agent {
   kubernetes {
     yaml """
       kind: Pod
       metadata:
         name: kaniko
       spec:
         containers:
         - name: jnlp
           workingDir: /home/jenkins
         - name: kaniko
           workingDir: /home/jenkins
           image: gcr.io/kaniko-project/executor:debug
           command:
           - /busybox/cat
           tty: true
           volumeMounts:
           - name: docker-config
             mountPath: /kaniko/.docker/
           - name: aws-secret
             mountPath: /root/.aws/
         restartPolicy: Never
         volumes:
         - name: docker-config
           configMap:
             name: docker-config 
         - name: aws-secret
           secret:
             secretName: aws-secret
"""
   }
 }
 stages {
   stage('Build DockerImage with Kaniko') {
     environment {
       PATH = "/busybox:/kaniko:$PATH"
     }
     steps {
              withAWS(credentials:'bartu-ecr', roleAccount:'130575395405', role:'arn:aws:iam::130575395405:role/talent_role') {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                        # Install the Amazon ECR Credential Helper
                        wget -O /kaniko/acr-cred-helper https://amazon-ecr-credential-helper-releases.s3.us-west-2.amazonaws.com/0.5.0/linux-amd64/docker-credential-ecr-login
                        chmod +x /kaniko/acr-cred-helper
                        mv /kaniko/acr-cred-helper /kaniko/docker-credential-ecr-login

                        # Use the ECR Credential Helper to authenticate
                        /kaniko/docker-credential-ecr-login list
                        /kaniko/docker-credential-ecr-login get

                        # Running Kaniko build
                        /kaniko/executor --context `pwd` --dockerfile worker/Dockerfile --verbosity debug --destination 130575395405.dkr.ecr.us-east-1.amazonaws.com/worker:latest
                    '''
                }
              }
     }
   }
 }
}
