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
  agent none
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
      name: 'OTHER_SERVICE_GIT_BRANCH',
      defaultValue: 'master',
      description: 'Deafult Git branch'
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
        VERSION = gitgetCommitHash()
        DOCKER_BUILD_TAG = "latest"
        PROJECT_Name = "my-first-java-code"
        GIT_CREDENTIAL_ID = ""
      }
      stages {
        stage ('Git checkout') {
          steps {
            checkout scm
            dir('simple-java-maven-app') {
              cloneComponent(SIMPLE_JAVA_MAVEN_APP, OTHER_SERVICE_GIT_BRANCH, GIT_CREDENTIAL_ID)
             }
          } // steps
        } // stage checkout
        stage('Build Java App') {
          steps {
            dir('simple-java-maven-app') {
              sh """
                mvn -B -DskipTests clean package
              """
            }
          }
        } //stage('Build Java App')
      } // stages
    } // stage Build on master
    stage('Deploy VM'){
      when {
        expression { params.BUILD_Type ==~ /build_and_test|deploy/ }
      }
          steps {
            script {
              if ( env.StackExistsSatus  ==~ /CREATE_COMPLETE|UPDATE_COMPLETE/ ) {
                sh """
                  echo "--=#####   Placeholder to completed   #####=--"
                """
              } else {
                sh """
                  echo "--=#####   Placeholder to create   #####=--"
                """
              }
            }
          } // steps
    } // stage Deploy VM
      stage('Run tests on VM') {
        when {
            expression { params.BUILD_Type == 'deploy' }
          }
        steps {
          script {
              sh """
                echo "--=#####   Waiting for VM   #####=--"
                curl -s -S "http://$Project_DNS_Name:$CONFIG_SERVER_Port$CO_SE_TGHealthCheckPath" > /dev/null
              """
          }
        } // steps
      } // stage Run tests on VM
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
