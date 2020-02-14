timestamps{
    node('nodejs'){
        stage('Checkout') {
            checkout([$class: 'GitSCM', branches: [[name: '*/openshift']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/bernardtm/jenkins-shared-library.git']]])
            // checkout scm
        }
        stage('Compile') {
          this.jenkinsContext.sh 'npm install'
        }
        stage('Test') {
          this.jenkinsContext.sh 'npm test'
        }
        stage('Build') {
          this.jenkinsContext.print('Build')
        }
    }
}