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

                        # Running Kaniko build
                        /kaniko/executor --context `pwd` --dockerfile vote/Dockerfile --verbosity debug --destination 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest
                    '''
                }
              }
     }
   }
 }
}
