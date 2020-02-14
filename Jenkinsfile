timestamps{
  def PROJECT = 'nodejs-backend-teste'
    node('nodejs'){
        stage('Checkout') {
            // checkout([$class: 'GitSCM', branches: [[name: '*/openshift']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/bernardtm/jenkins-shared-library.git']]])
            checkout scm
        }
        stage('Compile') {
          sh 'npm install'
        }
        stage('Test') {
          sh 'npm test'
        }
        openshift.withCluster() {
            openshift.withProject("${PROJECT}-qa") {
                stage('Build'){
                  echo "${PROJECT}"
                    // if (!openshift.selector("bc", "${NAME}").exists()) {
                    //     echo "Criando build"
                    //     def nb = openshift.newBuild(".", "--strategy=source", "--image-stream=${IMAGE_BUILDER}", "--name=${NAME}", "-l app=${LABEL}")
                    //     def buildSelector = nb.narrow("bc").related("builds")
                    //     buildSelector.logs('-f')
                    // }//if
                    // else {
                    //     echo "Build já existe. Iniciando build"
                    //     def build = openshift.selector("bc", "${NAME}").startBuild()
                    //     build.logs('-f')
                    // }//else
                    }//stage
                // stage('Deploy QA') {
                //     echo "Criando Deployment"
                //     openshift.apply(openshift.process(readFile(file:"${TEMPLATE}-qa.yml"), "--param-file=template_environments"))
                //     openshift.selector("dc", "${NAME}").rollout().latest()
                //     def dc = openshift.selector("dc", "${NAME}")
                //     dc.rollout().status()
                // }//stage
            }//withProject
        }//withCluster
    }
}