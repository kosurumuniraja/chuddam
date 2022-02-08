pipeline {

  agent any

environment {

      sonar_url = 'http://172.31.35.245:9000'
      sonar_username = 'admin'
      sonar_password = 'admin'
      nexus_url = '35.222.210.226:8081'
      artifact_version = '0.0.1'

 }
  options {
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '2', numToKeepStr: '10'))
    ansiColor('xterm')
    disableConcurrentBuilds()
  }


  tools {
    jdk 'JAVA8'
    maven 'Maven-3.3.9'

  }


  parameters {
      string(defaultValue: 'discovery-service', description: 'COC Service name', name: 'Service')
      string(defaultValue: 'master', description: 'COC branch to build', name: 'Branch')
      choice(
        name: 'Cluster',
        choices: ['gke_cognitive-genie-63754_us-central1-a_rapid-istio'],
        description: 'Cluster you want to deploy the pod to' )

  }


  stages {
    stage ('Cleanup Workspace') {
      steps {
        cleanWs()
        }
      }
    stage ('Checkout Codebase') {
        steps {
          echo "====== Current WORKSPACE Directory is ${env.WORKSPACE} ======="
          echo "============== GIT Source Code Branch: ${Branch} =============="
          checkout(
            [
              $class: 'GitSCM',
              branches: [[name: '${Branch}']],
              doGenerateSubmoduleConfigurations: false,
              extensions: [
                [$class: 'CleanBeforeCheckout'],
                [$class: 'CloneOption', depth: 0, noTags: false, reference: '', shallow: false, timeout: 120],
                [$class: 'CheckoutOption', timeout: 120],
                [$class: 'CleanBeforeCheckout'],
                [$class: 'CleanCheckout'],
                [$class: 'PruneStaleBranch']
                ],
                submoduleCfg: [],
                userRemoteConfigs: [
                  [
                    credentialsId: '5a2a0d63-2d2c-4bb0-94a0-db4dc740f911',
                    url: 'https://pscode.lioncloud.net/commerceoncloud/${Service}.git'
                    ]
                  ]
                ],
              )
          }
        }
     stage ('Build and Compile Codebase') {
        steps {
          sh '''
          mvn clean install -U -Dmaven.test.skip=true
          mvn -f ${WORKSPACE}/ clean install checkstyle:checkstyle findbugs:findbugs pmd:pmd pmd:cpd cobertura:cobertura -Dcobertura.report.format=xml
          '''
        }
      }
    stage ('adding check for API contract testing'){
         steps{
          script{
              if (fileExists ("${WORKSPACE}/src/test/resources/contracts/**/*.groovy" )) {
              currentBuild.result = 'CONTINUE'
              }else{
              currentBuild.result = 'ABORTED'
              print "Please add APIContract testing files"
             }
           }
         }
       }
     
     stage ('Sonarqube Analysis'){
           steps {
           withSonarQubeEnv('SonarQube') {
           sh '''
           mvn clean package org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=false
           mvn -e -B sonar:sonar -Dsonar.java.source=1.8 -Dsonar.host.url="${sonar_url}" -Dsonar.login="${sonar_username}" -Dsonar.password="${sonar_password}" -Dsonar.sourceEncoding=UTF-8
           '''
           }
         }
      }
