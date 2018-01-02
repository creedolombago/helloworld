#!/usr/bin/groovy

podTemplate(label: 'k8s-jnlp', name: 'k8s-jnlp', containers: [
    containerTemplate(name: 'jnlp', image: 'jenkinsci/jnlp-slave:3.10-1-alpine', args: '${computer.jnlpmac} ${computer.name}', workingDir: '/home/jenkins', nodeSelector: 'beta.kubernetes.io/instance-type=r3', resourceRequestCpu: '200m', resourceLimitCpu: '200m', resourceRequestMemory: '1024Mi', resourceLimitMemory: '1024Mi'),
    containerTemplate(name: 'docker', image: 'docker:17.06.0-ce', privileged: true, command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.7.3', command: 'cat', ttyEnabled: true)
],
volumes:[
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
]){


  node ('k8s-jnlp') {
  
    checkout scm
      
    try {
        
        stage ('docker build') {

          container('docker') {
 
                sh """
                  docker build --rm -t localhost:5000/helloworld .
                """
          }

        } // 'docker build'

        stage ('docker push') {

          container('docker') {
 
                sh """
                  docker push localhost:5000/helloworld
                """
          }

        } // 'docker push'

        stage ('kubectl deploy') {

          container('kubectl') {
 
                sh """
                  kubectl run helloworld --image=localhost:5000/helloworld --replicas=2 --port=80
                  kubectl expose deployment helloworld --port=80 --type=NodePort 
                """
          }

        } // 'deploy'



    } //try
    finally {
      
      if (fileExists('build/test-results/test')) {
        junit 'build/test-results/test/*.xml'
      }
    }
  }    
}
