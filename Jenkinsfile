#!/usr/bin/env groovy

import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonSlurper
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

env.AWS_ECR_LOGIN=true

def String LRL_RSIT_ENV_Parameters

def CONFIG_SERVER
def BuildUserID
def String StackExists = ""
def String OTHER_SERVICE_GIT_BRANCH

//Git Repositories
def String CONFIG_SERVER_URL = "https://github.com/EliLillyCo/LRL_Research_Data_Config_Server.git"
def String COMMON_INFRA_REPO_URL = "https://github.com/EliLillyCo/LRL_Research_Data_Common_Infrastructure.git"

// Services
def String Service_CF_File = "CI_CD/Blueprint/Lilly/Service_for_ECS.template"
def String ECR_CF_File = "CI_CD/Blueprint/ecr.template"
def String CONFIG_SERVER_Service_Tag = "config-server"
def String CONFIG_SERVER_CPU = "512"
def String CONFIG_SERVER_RAM = "512"
def String CO_SE_TGHealthCheckPath = '\"/actuator/health\"'

def Boolean errorHandling = false
def String stageTestCodeBuildError
def String stageTestMasterError

@Library('joblib@develop') _

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
      name: 'Spring_Cloud_Config_Server_Git_URI',
      defaultValue: 'https://github.com/EliLillyCo/LRL_Research_Data_Config_Server_Configs.git',
      description: 'Git repository base url'
    )
    choice(
      choices: ['YES','NO'],
      name: 'SONARQUBE_ANALYSIS',
      description: 'YES: Run analysis, NO: Skip analysis' 
    )
    string(
      name: 'SONARQUBE_URL',
      defaultValue: 'http://dictionary.dev.data.lrl.lilly.com:9091',
      description: 'SonarQube endpoint'
    )
    string(
      name: 'SONAR_BRANCH_NAME',
      defaultValue: 'master',
      description: 'Analysed Branch Name (First run have to be master)'
    )
  } // parameters

  stages {
    stage('Check branches DEV'){
      when {
        expression { params.TARGET_AWS_ENV == 'DEV' }
      }
      steps {
        script {
          OTHER_SERVICE_GIT_BRANCH = "release"
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
        LRL_RSIT_ENV_Parameters = getParameterFromStore('LRL_RSIT_ENV_Parameters')
        json = parseJsonToMap(LRL_RSIT_ENV_Parameters)
        VERSION = gitgetCommitHash()
        Service_CF_FilePath = "$env.WORKSPACE/common_infrastructure/$Service_CF_File"
        ECR_CF_FilePath = "$env.WORKSPACE/common_infrastructure/$ECR_CF_File"
        AWS_AccountNumber = GetAWSAccountID()
        ECR_Base = "$AWS_AccountNumber" + ".dkr.ecr.us-east-2.amazonaws.com"
        PROJECT_Name = "$json.Infrastructure.DictionaryName"
        GIT_CREDENTIAL_ID = "$json.Infrastructure.GitCredentialID"
        ECR_Base_URL ="http://" + "$ECR_Base" + "/"
        CONFIG_SERVER_STACK_NAME = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag" + "-service"
        ECR_repositoryName_CO_SE = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag"
        ECR_IMAGE = "$ECR_repositoryName_CO_SE:$VERSION"
        ECRURL = "$ECR_Base_URL" + "$ECR_repositoryName_CO_SE"
        ECS_StackName  = "$PROJECT_Name" + "-ECS"
        CONFIG_SERVER_ImageUrl = "$ECR_Base" + "/" + "$ECR_IMAGE"
        ECR_CONFIG_SERVER_STACK_NAME = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag" + "-ecr"
        ECS_StackExistsSatus = checkAWSStack(env.ECS_StackName)
        CONFIG_SERVER_StackExistsSatus = checkAWSStack(env.CONFIG_SERVER_STACK_NAME)
        ECR_CONFIG_SERVER_StackExistsStatus = checkAWSStack(env.ECR_CONFIG_SERVER_STACK_NAME)
        PREV_VERSION = getPreviousVersion(env.CONFIG_SERVER_STACK_NAME)
      }
      stages {
        stage ('Git checkout') {
          steps {
            checkout scm
            dir('common_infrastructure') {
              cloneComponent(COMMON_INFRA_REPO_URL, OTHER_SERVICE_GIT_BRANCH, GIT_CREDENTIAL_ID)
             }
          }
        }
      } // stages
    } // Build on master
    stage('Build on codebuild'){
      agent { label "codebuilder-config-server" }
      environment {
        LRL_RSIT_ENV_Parameters = getParameterFromStore('LRL_RSIT_ENV_Parameters')
        json = parseJsonToMap(LRL_RSIT_ENV_Parameters)
        VERSION = gitgetCommitHash()
        DOCKER_BUILD_TAG = "codebuild"
        Service_CF_FilePath = "$env.WORKSPACE/common_infrastructure/$Service_CF_File"
        ECR_CF_FilePath = "$env.WORKSPACE/common_infrastructure/$ECR_CF_File"
        AWS_AccountNumber = GetAWSAccountID()
        ECR_Base = "$AWS_AccountNumber" + ".dkr.ecr.us-east-2.amazonaws.com"
        PROJECT_Name = "$json.Infrastructure.DictionaryName"
        GIT_CREDENTIAL_ID = "$json.Infrastructure.GitCredentialID"
        ECR_Base_URL ="http://" + "$ECR_Base" + "/"
        CONFIG_SERVER_STACK_NAME = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag" + "-service"
        ECR_repositoryName_CO_SE = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag"
        ECR_IMAGE = "$ECR_repositoryName_CO_SE:$VERSION"
        ECRURL = "$ECR_Base_URL" + "$ECR_repositoryName_CO_SE"
        ECS_StackName  = "$PROJECT_Name" + "-ECS"
        CONFIG_SERVER_ImageUrl = "$ECR_Base" + "/" + "$ECR_IMAGE"
        ECR_CONFIG_SERVER_STACK_NAME = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag" + "-ecr"
        ECS_StackExistsSatus = checkAWSStack(env.ECS_StackName)
        CONFIG_SERVER_StackExistsSatus = checkAWSStack(env.CONFIG_SERVER_STACK_NAME)
      }
      stages {
        stage ('Git checkout') {
          steps {
            checkout scm
            dir('common_infrastructure') {
              cloneComponent(COMMON_INFRA_REPO_URL, OTHER_SERVICE_GIT_BRANCH, GIT_CREDENTIAL_ID)
             }
          } // steps
        } // stage checkout

        stage('Build Config Server') {
          steps {
            dir('configserver') {
              sh """
                mvn -q clean install
              """
            }
          }
        } //stage('Build Config Server')

        stage('Build Config Server docker image') {
          steps {
            script {
                CONFIG_SERVER = docker.build("$DOCKER_BUILD_TAG-config-server", "-f ./configserver/Dockerfile ./configserver/")
            }
          }
        } // stage('Build Config Server docker image')

        stage('Start docker containers') {
          when {
            expression {
              params.BUILD_Type == 'build_and_test' ||
              params.BUILD_Type == 'deploy'
            }
          }
          steps {
            withCredentials([usernamePassword(credentialsId: 'event_cordinator_config_server', passwordVariable: 'Spring_Cloud_Config_Server_Git_Password', usernameVariable: 'Spring_Cloud_Config_Server_Git_Username')])
            {
            script {
              println "Start containers"
              sh """
                sysctl -w vm.max_map_count=262144
                docker run -h $DOCKER_BUILD_TAG-config-server --name $DOCKER_BUILD_TAG-config-server -p 7091:7091 \
                 -e AWS_REGION=us-east-2  \
                 -e SPRING_CLOUD_CONFIG_SERVER_GIT_URI=$Spring_Cloud_Config_Server_Git_URI \
                 -e SPRING_CLOUD_CONFIG_SERVER_GIT_USERNAME=$Spring_Cloud_Config_Server_Git_Username \
                 -e SPRING_CLOUD_CONFIG_SERVER_GIT_PASSWORD=$Spring_Cloud_Config_Server_Git_Password \
                 -d -t $DOCKER_BUILD_TAG-config-server:latest
              """
            }
          } //withCredentials
        }
      } // stage('Start docker containers')

      stage('Run Config server tests') {
        when {
          expression { params.BUILD_Type ==~ /build_and_test|deploy/ }
        }
        steps {
          script {
            try { 
              sh """
                echo "THIS IS JUST A PLACEHOLDER for test cases."
                #We are aiting for the test case.
              """
            }catch(error){
                stageTestMasterError = env.STAGE_NAME
                errorHandling = true
                println(error)
                unstable('CONFIG SERVER TESTS ARE FAILED!')
            }
          }
        }
      } // stage('Run tests')
      stage('Run SonarQube Analysis'){
          steps{
            dir("configserver"){
              script{
                if( "$SONARQUBE_ANALYSIS" == "YES" ){ 
                  sh """
                    mvn sonar:sonar -Dsonar.host.url="${params.SONARQUBE_URL}" \
                                    -Dsonar.projectName="DATAPIPE Config Server" \
                                    -Dsonar.projectKey="lilly.cs:config-server" \
                                    -Dsonar.branch.name="${params.SONAR_BRANCH_NAME}" \
                                    -Dsonar.coverage.jacoco.xmlReportPaths="target/site/jacoco/jacoco.xml"
                  """
                }else{
                  echo "Skip SonarQube Analysis"
                }
              }
            }
          }
        }
      stage('Clean up') {
        when {
          expression { params.BUILD_Type ==~ /build_and_test|deploy/ }
        }
        steps {
          script {
            dir('container_logs') {
              sh """
                AllContainers=\$(docker ps -a --format '{{.Names}}')
                for C_NAME in \$AllContainers; do docker logs \$C_NAME > \$C_NAME-container.log && sleep 1 ; done
              """
            }
            sh "tar -c -f container_logs.tar container_logs"
          }
          archiveArtifacts artifacts: 'container_logs.tar', fingerprint: true
          copyArtifacts filter: "container_logs.tar", fingerprintArtifacts: true, projectName: "${JOB_NAME}", selector: specific("${BUILD_NUMBER}"), target: 'container_logs'
        }
      } // stage('Clean up')

      stage('Build Config Server with DEV config') {
        when {
          expression { params.BUILD_Type == 'deploy' }
        }
        steps {
          script {
              CONFIG_SERVER = docker.build("$ECR_IMAGE", "-f ./configserver/Dockerfile ./configserver/")
          }
        }
      } // stage('Build Config Server with DEV config')

      stage ('Push to ECR') {
        when {
          expression { params.BUILD_Type == 'deploy' }
          //expression { params.TARGET_AWS_ENV ==~ /DEV|QA|PRD/ }
        }
        steps {
          script {
              withAWS(region:'us-east-2') {
                // login to ECR
                sh("eval \$(aws ecr get-login --region us-east-2 | sed 's|-e none||' | sed 's|https://||')")
              } // with asw
          }
          script {
            // Push the Docker image to ECR
              docker.withRegistry(ECRURL)
              {
                  docker.image(ECR_IMAGE).push()
              }
          }
        }
      } // Push to ECR
      stage('Push Config Server image to Artifactory') {
        when {
          allOf {
            expression { params.BUILD_Type == 'deploy' }
            expression { params.TARGET_AWS_ENV == 'DEV' }
          }
        }
        steps {
          withCredentials([
            usernamePassword(credentialsId: 'resd_artifactory_deploy', usernameVariable: 'ARTIFACTUSER', passwordVariable: 'ARTIFACTPASSWD')])  {
            script {
              sh """
                docker login elilillyco-datadict-sandbox-docker-lc.jfrog.io --username=$ARTIFACTUSER --password=$ARTIFACTPASSWD
              """
              println("Push Config Server image to Artifactory")
              sh """
                docker tag $ECR_IMAGE elilillyco-datadict-sandbox-docker-lc.jfrog.io/lrl-config-server:latest
                docker push elilillyco-datadict-sandbox-docker-lc.jfrog.io/lrl-config-server:latest
              """
            }
          } // with credentials
        }
      } // Stage (Push Config Server image to Artifactory)
      stage ('Delete codebuild logs') {
        steps {
          deleteOldBuildLogs()
        }
      } //stage ('Delete codebuild logs')
    } // stages
  } // stage agent codebuilder
  stage('Deploy from master'){
    when {
      expression { params.BUILD_Type ==~ /build_and_test|deploy/ }
    // expression { params.TARGET_AWS_ENV ==~ /DEV|QA|PRD/ }
    }
    agent { label "master" }
    environment {
      LRL_RSIT_ENV_Parameters = getParameterFromStore('LRL_RSIT_ENV_Parameters')
      json = parseJsonToMap(LRL_RSIT_ENV_Parameters)
      VERSION = gitgetCommitHash()
      Service_CF_FilePath = "$env.WORKSPACE/common_infrastructure/$Service_CF_File"
      ECR_CF_FilePath = "$env.WORKSPACE/common_infrastructure/$ECR_CF_File"
      AWS_AccountNumber = GetAWSAccountID()
      ECR_Base = "$AWS_AccountNumber" + ".dkr.ecr.us-east-2.amazonaws.com"
      PROJECT_Name = "$json.Infrastructure.DictionaryName"
      Project_DNS_Name = "$json.Infrastructure.ProjectDNSName"
      GIT_CREDENTIAL_ID = "$json.Infrastructure.GitCredentialID"
      CONFIG_SERVER_Port = "$json.Services.ConfigServer.Port"
      ECR_Base_URL ="http://" + "$ECR_Base" + "/"
      CONFIG_SERVER_STACK_NAME = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag" + "-service"
      ECR_repositoryName_CO_SE = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag"
      ECR_IMAGE = "$ECR_repositoryName_CO_SE:$VERSION"
      ECRURL = "$ECR_Base_URL" + "$ECR_repositoryName_CO_SE"
      ECS_StackName  = "$PROJECT_Name" + "-ECS"
      CONFIG_SERVER_ImageUrl = "$ECR_Base" + "/" + "$ECR_IMAGE"
      ECR_CONFIG_SERVER_STACK_NAME = "$PROJECT_Name" + "-" + "$CONFIG_SERVER_Service_Tag" + "-ecr"
      ECS_StackExistsSatus = checkAWSStack(env.ECS_StackName)
      CONFIG_SERVER_StackExistsSatus = checkAWSStack(env.CONFIG_SERVER_STACK_NAME)
      PREV_VERSION = getPreviousVersion(env.CONFIG_SERVER_STACK_NAME)
    }
    stages {
      stage('Extract artifacts') {
        steps {
          copyArtifacts filter: "container_logs.tar", fingerprintArtifacts: true, projectName: "${JOB_NAME}", selector: specific("${BUILD_NUMBER}"), target: 'container_logs'
          script {
            dir('container_logs') {
              sh """
               tar -xf container_logs.tar
              """
            }
          }
        }
      } // stage Clean up
      stage ('Deploy on ECS') {
        when {
          allOf {
            expression { env.ECS_StackExistsSatus ==~ /CREATE_COMPLETE|UPDATE_COMPLETE/ }
            expression { params.BUILD_Type == 'deploy' }
          }
        }
        steps {
          withCredentials([usernamePassword(credentialsId: 'event_cordinator_config_server', passwordVariable: 'Spring_Cloud_Config_Server_Git_Password', usernameVariable: 'SPRING_CLOUD_CONFIG_SERVER_GIT_USERNAME')])
          {
          script {
            def CO_SE_Service_Params = """\
              ParentClusterStack=${env.ECS_StackName} \
              ImageUrl=$CONFIG_SERVER_ImageUrl \
              ContainerPort=$CONFIG_SERVER_Port \
              ContainerCpu=$CONFIG_SERVER_CPU \
              ContainerMemory=$CONFIG_SERVER_RAM \
              ServiceName=$CONFIG_SERVER_Service_Tag \
              EnvVar1Name=SPRING_CLOUD_CONFIG_SERVER_GIT_URI \
              EnvVar1Value=$Spring_Cloud_Config_Server_Git_URI \
              EnvVar2Name=SPRING_CLOUD_CONFIG_SERVER_GIT_USERNAME \
              EnvVar2Value=$Spring_Cloud_Config_Server_Git_Username \
              EnvVar3Name=SPRING_CLOUD_CONFIG_SERVER_GIT_PASSWORD \
              EnvVar3Value=$Spring_Cloud_Config_Server_Git_Password \
              TGHealthCheckPath=$CO_SE_TGHealthCheckPath
            """

            if ( env.CONFIG_SERVER_StackExistsSatus  ==~ /CREATE_COMPLETE|UPDATE_COMPLETE/ ) {
              sh """
                set +x
                aws cloudformation deploy --region us-east-2 --template-file $Service_CF_FilePath --stack-name $CONFIG_SERVER_STACK_NAME \
                --no-fail-on-empty-changeset --capabilities CAPABILITY_IAM --parameter-overrides $CO_SE_Service_Params
                echo "--=#####   $CONFIG_SERVER_STACK_NAME completed   #####=--"
              """
            } else {
              sh """
                set +x
                aws cloudformation delete-stack --region us-east-2 --stack-name $CONFIG_SERVER_STACK_NAME
                aws cloudformation wait stack-delete-complete --region us-east-2 --stack-name $CONFIG_SERVER_STACK_NAME
                aws cloudformation deploy --region us-east-2 --template-file $Service_CF_FilePath --stack-name $CONFIG_SERVER_STACK_NAME \
                --no-fail-on-empty-changeset --capabilities CAPABILITY_IAM --parameter-overrides $CO_SE_Service_Params
                echo "--=#####   $CONFIG_SERVER_STACK_NAME completed   #####=--"
              """
            }
          }
        } // withCredentials
        }
      } // stage Deploy to ECS
      stage('Run tests on ECS') {
        when {
          allOf {
            expression { env.ECS_StackExistsSatus ==~ /CREATE_COMPLETE|UPDATE_COMPLETE/ }
            expression { params.BUILD_Type == 'deploy' }
          }
        }
        steps {
          script {
            try{
              sh """
                echo "Waiting for Service on ECS" && while ! nc -z $Project_DNS_Name $CONFIG_SERVER_Port < /dev/null > /dev/null 2>&1; do sleep 1 ;done && sleep 3
                echo "Check Service url" && curl -I "http://$Project_DNS_Name:$CONFIG_SERVER_Port$CO_SE_TGHealthCheckPath" && sleep 1
                curl -s -S "http://$Project_DNS_Name:$CONFIG_SERVER_Port$CO_SE_TGHealthCheckPath" > /dev/null
              """
            }catch(error){
                stageTestCodeBuildError = env.STAGE_NAME
                errorHandling = true
                println(error)
                unstable('CONFIG SERVER TEST ON ECS ENVIRONMENT ARE FAILED!')
            }
          }
        }
      } // stage Run tests
      stage('Error handling'){
        steps{
          script{
            println(errorHandling)
            if ( errorHandling == true ){
              if ( stageTestCodeBuildError == 'Run tests on ECS' && stageTestMasterError == 'Run Config server tests' ) {
                sh """
                  echo "Found ERROR in $stageTestCodeBuildError and $stageTestMasterError stage!"
                  exit 1
                """
              } else if ( stageTestCodeBuildError == 'Run tests on ECS' ) {
                sh """
                  echo "Found ERROR in $stageTestCodeBuildError stage!"
                  exit 1
                """
              } else if ( stageTestMasterError == 'Run Config server tests' ) {
                sh """
                  echo "Found ERROR in $stageTestMasterError stage!"
                  exit 1
                """
              } else {
                sh """
                  echo "Not found information about stage!"
                  exit 1
                """
              }
            } else {
                println("No ERROR Found!")
            }
          }
        }
      }//stage('Error handling')
    } // stages
  } // Deploy on master
  } //Stages

  // ToDo deploy from not develop/release/master into different port, deploy with SSL port
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

def notifyBuild(String buildStatus = 'STARTED') {
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} Build type: $BUILD_Type, target: ${params.BUILD_Type} (<${env.BUILD_URL}|Open Job>)"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)

}

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

def checkUser(String Job_Type) {
  def String AllowAWSDeploy
  wrap([$class: 'BuildUser']) {
    BuildUserID = sh ( script: 'echo "${BUILD_USER_ID}"', returnStdout: true).trim()
  }
  withCredentials([string(credentialsId: 'DictAllowed2DeployAWS',variable: 'Allowed2DeployAWS')]) {
    CheckAllowedUsers = Allowed2DeployAWS.contains(BuildUserID)
    if ( CheckAllowedUsers ) {
      AllowAWSDeploy = "ok"
    }
  }
  return AllowAWSDeploy
}

def checkAWSStack(STACKNAME){
  def String CheckStack
    StackExists = sh (script: "aws cloudformation list-stacks --region us-east-2 --output text --query \"StackSummaries[?StackName == '$STACKNAME'].StackStatus\" ", returnStdout: true).tokenize()
    CheckStack = StackExists[0]
  return CheckStack
}

def deleteOldBuildLogs(){
  CODE_BUILD_PROJECT_ID = env.NODE_NAME.split("\\.")[0]
  CODE_BUILD_IDS = sh (script: "aws codebuild list-builds-for-project --region us-east-2 --project-name $CODE_BUILD_PROJECT_ID --sort-order DESCENDING --output text --query \"ids[]\" ", returnStdout: true )
  DELETED_IDS = sh (script: " aws codebuild batch-delete-builds --region us-east-2 --ids $CODE_BUILD_IDS ", returnStdout: true )
  return DELETED_IDS
}

def RollbackAfterFailed(){
  if ( params.BUILD_Type == 'deploy' ) {
    CO_SE_Service_Params = """\
      ParentClusterStack=${env.ECS_StackName} \
      ImageUrl=$CONFIG_SERVER_ImageUrl \
      ContainerPort=$CONFIG_SERVER_Port \
      ContainerCpu=$CONFIG_SERVER_CPU \
      ContainerMemory=$CONFIG_SERVER_RAM \
      ServiceName=$CONFIG_SERVER_Service_Tag \
      EnvVar1Name=SPRING_CLOUD_CONFIG_SERVER_GIT_URI \
      EnvVar1Value=$Spring_Cloud_Config_Server_Git_URI \
      EnvVar2Name=SPRING_CLOUD_CONFIG_SERVER_GIT_USERNAME \
      EnvVar2Value=$Spring_Cloud_Config_Server_Git_Username \
      EnvVar3Name=SPRING_CLOUD_CONFIG_SERVER_GIT_PASSWORD \
      EnvVar3Value=$Spring_Cloud_Config_Server_Git_Password \
      TGHealthCheckPath=$CO_SE_TGHealthCheckPath
    """
    sh """
      echo "Rollback $CONFIG_SERVER_STACK_NAME Stack"
      aws cloudformation update-stack --region us-east-2 --template-body file://$Service_CF_FilePath --stack-name $CONFIG_SERVER_STACK_NAME --capabilities CAPABILITY_IAM --parameters $CO_SE_Service_Params
      aws cloudformation wait stack-update-complete --region us-east-2 --stack-name $CONFIG_SERVER_STACK_NAME
      echo "--=#####    $CONFIG_SERVER_STACK_NAME completed   #####=--"
    """
  }
}

@NonCPS
def parseJsonToMap(String json) {
    final slurper = new JsonSlurperClassic()
    return new HashMap<>(slurper.parseText(json))
}

def getPreviousVersion(PREVSTACKNAME){
  if ( env.CONFIG_SERVER_StackExistsSatus  ==~ /CREATE_COMPLETE|UPDATE_COMPLETE/ ) {
    PREV_VERSION_IMG = sh (script: "aws cloudformation describe-stacks --region us-east-2 --stack-name $PREVSTACKNAME --output text --query \"Stacks[*].Parameters[?ParameterKey=='ImageUrl'].ParameterValue\" ", returnStdout: true)
    return PREV_VERSION_IMG
  }
}
