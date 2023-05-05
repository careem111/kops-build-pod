 pipeline{
   agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          name: mypod
        spec:
          containers:
          - name: build-agent
            image: careem785/jenkins-build-agent:2.0
            command: 
             - cat
            tty: true
            volumeMounts:
            - name: dockersock
              mountPath: /var/run/docker.sock
          volumes:
          - name: dockersock
            hostPath:
              path: /var/run/docker.sock  
      '''
        }
   }
   tools {
        maven 'maven'
    }
    stages {
      stage('Checkout SCM') {
    	  steps{
          container('build-agent'){
            git branch: 'main',    url: 'https://github.com/careem111/loginapp-integration.git'
            sh 'mvn clean deploy -s settings.xml'
            }
	        }
        }      
      stage('Docker Build'){
        steps{
          container('build-agent'){
            script {
              docker.withRegistry( 'https://registry.hub.docker.com', 'docker'  ) {
              dockerImage = docker.build 'careem785/webapp_4-5-23'
              dockerImage.push('latest')
                  }
              }
            }
          }
      }
      stage('Helm Chart'){
        steps{
          container('build-agent'){
            dir('charts') {
              withCredentials([usernamePassword(credentialsId: 'jfrog', usernameVariable: 'username', passwordVariable: 'password')]) {
              sh '/usr/local/bin/helm package webapp'
              sh '/usr/local/bin/helm push-artifactory webapp-1.0.tgz https://kubekrm.jfrog.io/artifactory/dpthelm-helm-local/ --username $username --password $password'
                  }
             }
          }
        }
      }
    }
  }