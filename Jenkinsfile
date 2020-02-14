timestamps{
    node('nodejs'){

        stage('Checkout') {
            checkout scm
        }

        stage('Compile') {
          readProperties()
          sh 'npm install'
          sh "tar czf ${BUILD_CONFIG_NAME_VERSION}.tgz *"
          // sh 'npm test'
        }

        // def git_branch = sh (script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
        // println git_branch
        // if (git_branch ==~ 'master|HEAD') {
        //     stage ('Code Quality'){
        //         def sonar = load 'devops/sonar.groovy'
        //         sonar.codeQuality()
        //     }
        // }

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

              // stage('Tagging Image'){
              //   openshift.tag("${NAMESPACE}-qa/${BUILD_CONFIG_NAME_VERSION}:latest", "${PROJECT_IMAGES}/${BUILD_CONFIG_NAME_VERSION}:latest")
              // }

              stage('Deploy QA') {
                deploy("qa", "templateNode.yml")
              }
              stage('Proceed to HML') {
                  inputComTimeout(time: 20, unit: 'MINUTES', message: "Promover para Homolog?")
              }
            }//withProject
            def releaseType
            openshift.withProject("${NAMESPACE}-hml") {
                stage('Deploy HML') {
                    deploy("hml", "templateNode-hml-prd.yml")
                }
                stage('Proceed to Release') {
                    releaseType = inputComTimeout(
                            time: 3, unit: 'HOURS',
                            message: "Fechar versao?",
                            submitterParameter: 'submitter',
                            parameters: [[$class: 'hudson.model.ChoiceParameterDefinition',
                                          choices: 'major\nminor\npatch',
                                          description: 'Tipos de release',
                                          name: 'release']]
                    )
                }
            }
        }//withCluster
    }
}

def readProperties() {
    Map required = [
            APP_LABEL:                  '^\\S+$',
            PROJECT_NAME:               '^\\S+$',
            PROJECT_IMAGES:             '^\\S+$',
            BUILD_CONFIG_NAME_VERSION:  '^\\S+$',
            BUILD_CONFIG_NAME:          '^\\S+$',
            PORT:                       '^\\d+$',
    ]
    def props = readProperties file: 'jenkins.properties'
    if (props == null)
        error("Erro ao recuperar arquivo de propriedades!")
    if (required.keySet() != props.keySet())
        error("Propriedades no arquivo não condizem exatamente com as propriedades exigidas!")
    List<String> erros = []
    for (entry in props) {
        def value = ((String) entry.value).trim()
        if ( !( value ==~ required.get(entry.key) ) )
            erros.add("Valor da propriedade '" + entry.key + "' não está no formato esperado!")
        else
            env."${entry.key}" = value
    }
    if (!erros.isEmpty())
        error( erros.join("\n") )
}

def getVersion() {
    return readJSON(file: 'package.json').version
}

def addSummary(version, submitter) {
    manager.addBadge("package.gif", "Versao: ${version} Usuario: ${submitter}")
    manager.createSummary("package.gif")
            .appendText("<b>Versao:</b> ${version}<br><b>Usuario:</b> ${submitter}",
                    false, false, false, "black"
            )
}

def inputComTimeout(args = [:]) {
    try {
        return timeout(args) { return input(args) }
    } catch (e) { // org.jenkinsci.plugins.workflow.steps.FlowInterruptedException
        if ( e.causes?.get(0)?.getUser()?.toString() == 'SYSTEM' )
            println "Saída por timeout"
        throw e
    }
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

    def template = readFile file: "templates/${templateFile}"

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
    template = template.replace("\"@HEALTHCHECK@\"", "${HEALTHCHECK}")

    writeFile encoding: 'UTF-8', file: "template-${env}.yml", text: template
    println template
}

