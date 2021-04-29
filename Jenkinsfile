pipeline {
agent {
      label 'nodejs8'
  }
stages
  {
  stage ('install modules'){
   steps {
    sh '''
      npm install --verbose -d 
      npm install --save classlist.js
    '''
  }
 }
  stage ('Build App') {
  steps {
    sh '$(npm bin)/ng build --prod --build-optimizer'
  }
 }
  
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector("bc", "frontend").exists();
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newBuild("--name=frontend", "--image-stream=registry.redhat.io/rhscl/nodejs-4-rhel7", "--binary")
          }
        }
      }
    }
    
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.selector("bc", "frontend").startBuild("--from-dir=dist", "--wait")
          }
          sh 'echo "image build success"'
        }
      }
    }
   stage('Promote to DEV') {
      steps {
        script {
          openshift.withCluster() {
            openshift.tag("frontend:latest","frontend:prod")
          }
        }
      }
