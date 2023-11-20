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
                        /kaniko/executor --context `pwd`/vote --dockerfile `pwd`/vote/Dockerfile --verbosity debug --destination 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest
                    '''
                }
              }
     }
   }
   stage('Snyk scan') {
     steps {
      script{
        echo 'Testing...'
        //snykSecurity(
         // snykInstallation: 'snyk',
         // snykTokenId: 'snyk-token',
          // place other parameters here
        //)
        //snykSecurity container test 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest, snykInstallation: 'snyk', snykTokenId: 'snyk-token'
        snykSecurity snykInstallation: 'snyk', snykTokenId: 'snyk-token'
        def variable = sh(
                       script: 'snyk container test 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest --file=/home/jenkins/workspace/demo-pipeline/vote/Dockerfile', returnStatus: true)
        echo "error code = ${variable}"
        if (variable != 0) {
           echo " Alert for vulnerability found"
        }
       }
     }
   }
 }
}
