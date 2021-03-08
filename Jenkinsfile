//this is the scripted method with groovy engine
import hudson.model.Result

node {

    //Configuration
    def gitURL = "https://github.com/julg/testJenkinsFileFront.git"
    def gitCredentials = ''
    def jsDir = "dist/"
    def htmlBaseDir = "/var/www/html/licencesNationales/"
    def slackChannel = "#notif-licencesnationales"

    // Variables globales
    def ENV

    // Configuration du job Jenkins
    // On garde les 5 derniers builds par branche
    // On scanne les branches et les tags du Git
    properties([
            buildDiscarder(
                    logRotator(
                            artifactDaysToKeepStr: '',
                            artifactNumToKeepStr: '',
                            daysToKeepStr: '',
                            numToKeepStr: '5')
            ),
            parameters([
                    gitParameter(
                            branch: '',
                            branchFilter: 'origin/(.*)',
                            defaultValue: 'develop',
                            description: 'Sélectionner la branche ou le tag à déployer',
                            name: 'BRANCH_TAG',
                            quickFilterEnabled: false,
                            selectedValue: 'NONE',
                            sortMode: 'DESCENDING_SMART',
                            tagFilter: '*',
                            type: 'PT_BRANCH_TAG'),
                    choice(choices: ['DEV', 'TEST', 'PROD'], description: 'Sélectionner l\'environnement cible', name: 'ENV'),
            ])
    ])

    stage('Set environnement variables') {
        try {
            env.NODEJS_HOME = "${tool 'NodeJS 11.2.0'}"
            env.PATH="${env.NODEJS_HOME}/bin:${env.PATH}"
            sh 'npm --version'

            if (params.BRANCH_TAG == null) {
                throw new Exception("Variable BRANCH_TAG is null")
            } else {
                echo "Branch to deploy =  ${params.BRANCH_TAG}"
            }

            if (params.ENV == null) {
                throw new Exception("Variable ENV is null")
            } else {
                ENV = params.ENV
                echo "Target environnement =  ${ENV}"
            }

        } catch (e) {
            currentBuild.result = hudson.model.Result.NOT_BUILT.toString()
            notifySlack(slackChannel,e.getLocalizedMessage())
            throw e
        }
    }

    stage('SCM checkout') {
        try {
            checkout([
                    $class                           : 'GitSCM',
                    branches                         : [[name: "${params.BRANCH_TAG}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions                       : [],
                    submoduleCfg                     : [],
                    userRemoteConfigs                : [[credentialsId: "${gitCredentials}", url: "${gitURL}"]]
            ])

        } catch (e) {
            currentBuild.result = hudson.model.Result.FAILURE.toString()
            notifySlack(slackChannel,e.getLocalizedMessage())
            throw e
        }
    }

    stage('Information') {
        try {
            sh 'node -v'
            sh 'npm -v'

        } catch (e) {
           currentBuild.result = hudson.model.Result.FAILURE.toString()
           notifySlack(slackChannel,e.getLocalizedMessage())
           throw e
        }
    }


    stage('Dependencies') {
        try {
            sh 'npm install'
        } catch (e) {
            currentBuild.result = hudson.model.Result.FAILURE.toString()
            notifySlack(slackChannel,e.getLocalizedMessage())
            throw e
        }
    }

    stage('Build') {
        try {
            sh 'npm run build'
        } catch (e) {
            currentBuild.result = hudson.model.Result.FAILURE.toString()
            notifySlack(slackChannel,e.getLocalizedMessage())
            throw e
        }
    }

    stage ('deploy to raiponce'){
        try {
             if (ENV == 'DEV') {
                echo 'deployment on raiponce1-dev'
                sshagent(credentials: ['raiponce1-dev-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'develuser', passwordVariable: 'pass', usernameVariable: 'username')]) {
                        sh "ssh -tt devel@raiponce1-dev.v3.abes.fr \"cd ${htmlBaseDir} && rm -rf -d js && rm -rf -d css\""
                        sh "scp -r ${jsDir}*  ${username}raiponce1-dev.v3.abes.fr:${htmlBaseDir}"
                    }
                }

                echo 'deployment on raiponce2-dev'
                sshagent(credentials: ['raiponce2-dev-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'develuser', passwordVariable: 'pass', usernameVariable: 'username')]) {
                        sh "ssh -tt devel@raiponce2-dev.v3.abes.fr \"cd ${htmlBaseDir} && rm -rf -d js && rm -rf -d css\""
                        sh "scp -r ${jsDir}*  ${username}raiponce2-dev.v3.abes.fr:${htmlBaseDir}"
                    }
                }
            }
            if (ENV == 'TEST') {
                echo 'deployment on raiponce1-test'
                sshagent(credentials: ['raiponce1-test-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'develuser', passwordVariable: 'pass', usernameVariable: 'username')]) {
                        sh "ssh -tt devel@raiponce1-test.v3.abes.fr \"cd ${htmlBaseDir} && rm -rf -d js && rm -rf -d css\""
                        sh "scp -r ${jsDir}*  ${username}@raiponce1-test.v3.abes.fr:${htmlBaseDir}"
                    }
                }

                echo 'deployment on raiponce2-test'
                sshagent(credentials: ['raiponce2-test-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'develuser', passwordVariable: 'pass', usernameVariable: 'username')]) {
                        sh "ssh -tt devel@raiponce2-test.v3.abes.fr \"cd ${htmlBaseDir} && rm -rf -d js && rm -rf -d css\""
                        sh "scp -r ${jsDir}*  ${username}@raiponce2-test.v3.abes.fr:${htmlBaseDir}"
                    }
                }
            }
            if (ENV == 'PROD') {
                echo 'deployment on raiponce1-prod'
                sshagent(credentials: ['raiponce1-prod-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'develuser', passwordVariable: 'pass', usernameVariable: 'username')]) {
                        sh "ssh -tt devel@raiponce1.v3.abes.fr \"cd ${htmlBaseDir} && rm -rf -d js && rm -rf -d css\""
                        sh "scp -r ${jsDir}*  ${username}@raiponce1.v3.abes.fr:${htmlBaseDir}"
                    }
                }

                echo 'deployment on raiponce2-prod'
                sshagent(credentials: ['raiponce2-prod-ssh-key']) {
                    withCredentials([usernamePassword(credentialsId: 'develuser', passwordVariable: 'pass', usernameVariable: 'username')]) {
                        sh "ssh -tt devel@raiponce2.v3.abes.fr \"cd ${htmlBaseDir} && rm -rf -d js && rm -rf -d css\""
                        sh "scp -r ${jsDir}*  ${username}@raiponce2.v3.abes.fr:${htmlBaseDir}"
                    }
                }
            }

        } catch(e) {
            currentBuild.result = hudson.model.Result.FAILURE.toString()
            notifySlack(slackChannel,e.getLocalizedMessage())
            throw e
        }
    }

    currentBuild.result = hudson.model.Result.SUCCESS.toString()
    notifySlack(slackChannel,"Congratulation !")
}

def notifySlack(String slackChannel, String info = '') {
    def colorCode = '#848484' // Gray

    switch (currentBuild.result) {
        case 'NOT_BUILT':
            colorCode = '#FFA500' // Orange
            break
        case 'SUCCESS':
            colorCode = '#00FF00' // Green
            break
        case 'UNSTABLE':
            colorCode = '#FFFF00' // Yellow
            break
        case 'FAILURE':
            colorCode = '#FF0000' // Red
            break;
    }

    String message = """
        *Jenkins Build*
        Job name: `${env.JOB_NAME}`
        Build number: `#${env.BUILD_NUMBER}`
        Build status: `${currentBuild.result}`
        Branch or tag: `${params.BRANCH_TAG}`
        Target environment: `${params.ENV}`
        Message: `${info}`
        Build details: <${env.BUILD_URL}/console|See in web console>
    """.stripIndent()

    return slackSend(tokenCredentialId: "slack_token",
            channel: "${slackChannel}",
            color: colorCode,
            message: message)
}

