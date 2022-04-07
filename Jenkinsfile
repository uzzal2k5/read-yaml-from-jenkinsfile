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
        configData = parser.load(properties)
        appName = config.configSet?."${configParam}"?.appApplicationName
        controlHost = config.configSet?."${configParam}"?.appControllerHostName
        nodeName = config.configSet?."${configParam}"?.appNodeName
        appTier = config.configSet?."${configParam}"?.appTierName

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
                    'Configuration Version 0',
                    'Configuration Version 1',
                    'Configuration Version 2'
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
                GitClone(REPOSITORY_URL,'main')
            }
        }
        stage("Call Param Values"){
            steps {
                PARAM = ("${params.configParam}" != null ) ? "${params.configParam}": "configVersion0"
                CONFIG_FILE = readFile "${env.WORKSPACE}/config/config-v1.0.yml"
                //final (String appName, String controlHost, String nodeName, String appTier) = getProperties(CONFIG_FILE,PARAM)

                
            }
        }
    }
}
