#!/usr/bin/env groovy

import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonSlurper
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

def String OTHER_SERVICE_GIT_BRANCH

//Git Repositories
def String SIMPLE_JAVA_MAVEN_APP = "https://github.com/tamaslipka/simple-java-maven-app.git"

//Error Handling
def Boolean errorHandling = false
def String stageTestCodeBuildError
def String stageTestMasterError


pipeline {
  agent { label "master" }
  tools {
    git 'Default'
    maven 'Maven'
  }
  parameters {
    choice(
      choices: ['build','build_and_test','deploy'],
      name: 'BUILD_Type',
      description: 'Select what do you want to run: build, build_and_test, deploy'
    )
    choice(
      choices: ['DEV','QA','PRD'],
      name: 'TARGET_AWS_ENV',
      description: 'Name of the target AWS environment'
    )
    string(
      name: 'NEW_VM_NAME',
      defaultValue: '',
      description: 'Deafult name of the new VM'
    )
    string(
      name: 'OTHER_SERVICE_GIT_BRANCH',
      defaultValue: 'master',
      description: 'Deafult Git branch'
    )
    string(
      name: 'GIT_CREDENTIAL_ID',
      defaultValue: '',
      description: 'Deafult Git credential'
    )
  } // parameters

  stages {
    stage('Check branches'){
      when {
        expression { params.TARGET_AWS_ENV == 'DEV' }
      }
      steps {
        script {
          OTHER_SERVICE_GIT_BRANCH = "develop"
        } // script
      } // steps
    } // stage('Check branches DEV')
    stage('Check branches QA'){
      when {
        expression { params.TARGET_AWS_ENV == 'QA' }
      }
      steps {
        script {
          OTHER_SERVICE_GIT_BRANCH = "release"
        } // script
      } // steps
    } // stage('Check branches QA')
    stage('Check branches PRD'){
      when {
        expression { params.TARGET_AWS_ENV == 'PRD' }
      }
      steps {
        script {
          OTHER_SERVICE_GIT_BRANCH = "master"
        } // script
      } // steps
    } // stage('Check branches PRD')
    stage('Build on master'){
      agent { label "master" }
      environment {
        DOCKER_BUILD_TAG = "latest"
        PROJECT_Name = "my-first-java-code"
      }
      stages {
        stage ('Git checkout') {
          steps {
            checkout scm
          } // steps
        } // stage checkout
      } // stages
    } // stage Build on master
    stage('Deploy VM'){
      when {
        expression { params.BUILD_Type ==~ /build_and_test|deploy/ }
      }
          steps {
            script {
              sh """
                  az group create --name $NEW_VM_NAME-ResourceGroup --location eastus
                  az vm create --resource-group $NEW_VM_NAME-ResourceGroup --name $NEW_VM_NAME --image UbuntuLTS --admin-username azureuser --generate-ssh-keys
                  az vm open-port --port 80 --resource-group $NEW_VM_NAME-ResourceGroup --name $NEW_VM_NAME
                """
            } // script
          } // steps
    } // stage Deploy VM
      stage('Run tests on VM') {
        when {
            expression { params.BUILD_Type == 'deploy' }
          }
        steps {
          script {
              sh """
                echo "--=#####   The new VM public IP is:   #####=--"
                az vm show -d -g $NEW_VM_NAME-ResourceGroup -n $NEW_VM_NAME --query publicIps -o tsv
              """
          }
        } // steps
      } // stage Run tests on VM
      stage('Install Nginx on VM') {
        when {
            expression { params.BUILD_Type == 'deploy' }
          }
        steps {
          script {
              sh """
                echo "--=#####   Install Nginx on new VM is in progress  #####=--"
                az vm run-command invoke -g $NEW_VM_NAME-ResourceGroup -n $NEW_VM_NAME --command-id RunShellScript --scripts "sudo apt-get -y update && sudo apt-get install -y nginx"
                echo "--=#####   Install Nginx on new VM is done  #####=--"
              """
          }
        } // steps
      } // stage Install Nginx on VM
  } // Stages
  post {
    failure {
      node('master') {
        //RollbackAfterFailed()
      }
    }
    always {
      node('master') {
        //notifyBuild(currentBuild.result)
      }
    }
  }
} // pipeline

def cloneComponent(repoUrl, repoBranch, credentialId) {
  checkout([
    $class: 'GitSCM',
    branches: [[name: repoBranch]],
    extensions: [
      [$class: 'CleanBeforeCheckout']
    ],
    userRemoteConfigs: [[
      url: repoUrl,
      credentialsId: credentialId,
    ]]
  ])
}

def RollbackAfterFailed(){
  if ( params.BUILD_Type == 'deploy' ) {
    sh """
      echo "Rollback VM process started"
      echo "-=== This is just a placeholder ===-"
      echo "Rollback VM process completed"
    """
  }
}
