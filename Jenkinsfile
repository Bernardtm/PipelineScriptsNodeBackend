timestamps{
    node('nodejs'){
        stage('Checkout') {
            // checkout([$class: 'GitSCM', branches: [[name: '*/openshift']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/bernardtm/jenkins-shared-library.git']]])
            checkout scm
        }
        stage('Compile') {
          readProperties()
          sh 'npm install'
          sh "tar czf ${BUILD_CONFIG_NAME_VERSION}.tgz *"
          // sh 'npm test'
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
              stage('Deploy QA') {
                deploy("qa", "devops/templateNode.yml")
              }
            }//withProject
        }//withCluster
    }
}

def readProperties() {    
    def props = readProperties file: 'jenkins.properties'
    if (props != null && props.size() > 0)
        propsToEnv(props)
    else
        error("Erro ao recuperar arquivo de propriedades!")
}

def propsToEnv(props) {
    for (entry in props) {
        env."${entry.key}" = entry.value
    }
}

def deploy(env, templateFile) {
    println "Criando Deployment - " + env

    mountTemplate(env, templateFile)

    openshift.apply(openshift.process(readFile(file:"template-${env}.yml")))
    openshift.selector("dc", "${BUILD_CONFIG_NAME_VERSION}").rollout().latest()
    def dc = openshift.selector("dc", "${BUILD_CONFIG_NAME_VERSION}")
    dc.rollout().status()
}

def mountTemplate(env, templateFile) {

    def template = readFile file: "${templateFile}"

    template = template.replace("@APP_LABEL@", "${APP_LABEL}")
    template = template.replace("@PROJECT_NAME@", "${PROJECT_NAME}")
    template = template.replace("@PROJECT_IMAGES@", "${PROJECT_IMAGES}")
    template = template.replace("@BUILD_CONFIG_NAME_VERSION@", "${BUILD_CONFIG_NAME_VERSION}")
    template = template.replace("@BUILD_CONFIG_NAME@", "${BUILD_CONFIG_NAME_VERSION}")
    //template = template.replace("@BUILD_CONFIG_NAME@", "${BUILD_CONFIG_NAME}")
    template = template.replace("@ENVIRONMENT@", env)
    template = template.replace("@VERSION@", "latest")
    template = template.replace("\"@MAX_REPLICAS@\"", "3")
    template = template.replace("\"@MIN_REPLICAS@\"", "1")
    template = template.replace("\"@PORT@\"", "${PORT}")

    writeFile encoding: 'UTF-8', file: "template-${env}.yml", text: template
    println template
}

