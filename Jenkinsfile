#!/usr/bin/env groovy
import groovy.json.*
import groovy.json.JsonSlurperClassic
import org.yaml.snakeyaml.Yaml

def REPOSITORY_URL = "git@github.com:uzzal2k5/read-yaml-from-jenkinsfile.git"

//Version & Release Specified Here
def getVersion(def projectJson){
    def slurper = new JsonSlurperClassic()
    project = slurper.parseText(projectJson)
    slurper = null
    return project.version.split('-')[0]
}

//Repository Clone from Git, Need to Set GIT_USERID in Jenkins
def GitClone(REPOSITORY,BRANCH ){
    def version, revision
    try {
        git(branch: "${BRANCH}",
                changelog: true,
                credentialsId: 'GIT_USERID',
                poll: true,
                url: "${REPOSITORY}"
        )
    }
    catch (Exception e) {
        println 'Some Exception happened here '
        throw e

    }
    finally {
        revision = version + "-" + sprintf("%04d", env.BUILD_NUMBER.toInteger())
        println "Start building  revision $revision"

    }
    return this
}
// Read Server Properties from config-v1.0.yml
def getProperties(String properties, String configParam){
    try {
        Yaml parser = new Yaml()
        def configData = parser.load(properties)
        def appName = config.configSet?."${configParam}"?.appApplicationName
        def controlHost = config.configSet?."${configParam}"?.appControllerHostName
        def nodeName = config.configSet?."${configParam}"?.appNodeName
        def appTier = config.configSet?."${configParam}"?.appTierName

        return [ appName, controlHost, nodeName,appTier ]
    }
    catch(Exception e){
        throw e
    }
    return  this
}


pipeline {
    agent any
    parameters {
            choice(
               description: "Choose Configuration Version",
               name: 'configParam',
               choices: [
                    'Select Configuration Version',
                    'configVersion0',
                    'configVersion1',
                    'configVersion2'
                     ]
               )
            string(name: 'STACK_NAME', defaultValue: 'example-stack', description: 'Enter the CloudFormation Stack Name.')
            string(name: 'PARAMETERS_FILE_NAME', defaultValue: 'example-stack-parameters.properties', description: 'Enter the Parameters File Name (Must contain file extension type *.properties)')
            credentials(name: 'CFN_CREDENTIALS_ID', defaultValue: '', description: 'AWS Account Role.', required: true)
            choice(
               description: "Select Repository Branch",
               name: 'Branch',
               choices: [
                    'dev',
                    'main'
                    ]
               )
            choice(
              name: 'REGION',
              choices: [
                  ' ',
                  'us-east-1',
                  'us-east-2'
                  ],
              description: 'AWS Account Region'
            )
            choice(
              name: 'ACTION',
              choices: ['create-changeset', 'execute-changeset', 'deploy-stack', 'delete-stack'],
              description: 'CloudFormation Actions'
            )
            booleanParam(name: 'TOGGLE', defaultValue: false, description: 'Are you sure you want to perform this action?')

    }
    stages {

        //Define Stages
        stage("Git Checkout"){
            steps {
                def BRANCH = ("${params.Branch}" != null ) ? "${params.Branch}": "main"
                GitClone(REPOSITORY_URL,BRANCH)
            }
        }
        stage("Call Param Values"){
            steps {
                def PARAM = ("${params.configParam}" != null ) ? "${params.configParam}": "configVersion0"
                def CONFIG_FILE = readFile "${env.WORKSPACE}/config/config-v1.0.yml"
                final def (String appName, String controlHost, String nodeName, String appTier) = getProperties(CONFIG_FILE,PARAM)

                echo "App Name =" appName
                echo "Control Host  Name =" controlHost
                echo "Node Name =" nodeName
                echo "App Tier =" appTier
            }
        }
    }
}
