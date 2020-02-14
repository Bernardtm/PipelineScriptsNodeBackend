timestamps{
  def NAMESPACE = 'node-backend'
  def BUILD_CONFIG_NAME_VERSION = 'node-backend-v2'
  def IMAGE_BUILDER = 'openshift/nodejs:8'
  def APP_LABEL = 'node-backend-v2'
  def TEMPLATE = 'template-nodejs'
    node('nodejs'){
        stage('Checkout') {
            // checkout([$class: 'GitSCM', branches: [[name: '*/openshift']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/bernardtm/jenkins-shared-library.git']]])
            checkout scm
        }
        stage('Compile') {
          sh 'npm install'
          sh "tar czf ${BUILD_CONFIG_NAME_VERSION}.tgz *"
          sh 'npm test'
        }
        openshift.withCluster() {
            openshift.withProject("${NAMESPACE}-qa") {
              stage('Build'){
                if (!openshift.selector("bc", "${BUILD_CONFIG_NAME_VERSION}").exists()) {
                    println "Criando build"
                    def nb = openshift.newBuild("--name=${BUILD_CONFIG_NAME_VERSION}", "--image-stream=${IMAGE_BUILDER}", "--binary", "-l app=${APP_LABEL}")
                    def buildSelector = openshift.selector("bc", "${BUILD_CONFIG_NAME_VERSION}").startBuild("--from-archive=${BUILD_CONFIG_NAME_VERSION}.tgz")
                    buildSelector.logs('-f')
                }
                else {
                    println("Build existente. Iniciando build")
                    def buildSelector = openshift.selector("bc", "${BUILD_CONFIG_NAME_VERSION}").startBuild("--from-archive=${BUILD_CONFIG_NAME_VERSION}.tgz")
                    buildSelector.logs('-f')
                }
              }
            }//withProject
        }//withCluster
    }
}