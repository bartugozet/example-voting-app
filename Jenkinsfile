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
       //SNYK_TOKEN = credentials('snyk-token')
        AWS_ACCESS_KEY_ID     = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
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

              withCredentials([[
                  $class: 'AmazonWebServicesCredentialsBinding', 
                  accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                  credentialsId: 'AWS_ACCOUNT'
              ]]) {
                    echo 'Testing...'
                    snykSecurity(
                      snykInstallation: 'snyk',
                      snykTokenId: 'snyk-token',
                      additionalArguments: '--container 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest --username=${accessKeyVariable} --password=${secretKeyVariable}'
                      //additionalArguments: '--docker 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest'
                    )        
              }
              // script {
              //       // Ensure 'snyk' is in the PATH
              //       def snykTool = tool 'snyk'
              //       withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
              //         withEnv(["PATH+SNYK=${snykTool}/bin", "SNYK_TOKEN=${SNYK_TOKEN}"]) {
              //             try {
              //                 sh "snyk auth ${SNYK_TOKEN}"
              //                 sh "snyk container test 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest"
              //             } catch (Exception e) {
              //                 echo "Snyk scan failed: ${e.message}"
              //                 currentBuild.result = 'FAILURE'
              //             }
              //         }
              //       }
              //   }
              // withAWS(credentials:'bartu-ecr', roleAccount:'130575395405', role:'arn:aws:iam::130575395405:role/talent_role') {
              // //script{
              //   echo 'Testing...'
              //   snykSecurity(
              //   snykInstallation: 'snyk',
              //   snykTokenId: 'snyk-token',
              //   additionalArguments: '--container 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest'
              //   //additionalArguments: '--docker 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest'
              //   )
              //   //sh ''' snyk container test 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest '''                
                
              //   //snykSecurity container test 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest, snykInstallation: 'snyk', snykTokenId: 'snyk-token'

              //   //snykSecurity snykInstallation: 'snyk', snykTokenId: 'snyk-token', additionalArguments: '--docker 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest --file=/home/jenkins/workspace/demo-pipeline/vote/Dockerfile'

              //   //def variable = sh(
              //     //             script: 'snyk container test 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest --file=/home/jenkins/workspace/demo-pipeline/vote/Dockerfile', returnStatus: true)
              //   //snyk container test 130575395405.dkr.ecr.us-east-1.amazonaws.com/vote:latest --file=/home/jenkins/workspace/demo-pipeline/vote/Dockerfile
              //   //echo "error code = ${variable}"
              //   //if (variable != 0) {
              //   //   echo " Alert for vulnerability found"
              //   //}
              // //}
              // }
     }
   }
  //  stage('Snyk scan') {
  //    steps {

  //    }
  //  }
 }
}
