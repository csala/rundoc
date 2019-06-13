pipeline {
  agent {
    docker { 
      image 'python:3.6'
       args '--user root:root --entrypoint=""'
    }
   }

  options {
    buildDiscarder(logRotator(numToKeepStr: '30', daysToKeepStr: '30'))
   }

stages {

    stage('Build rundoc') {
      steps {
            sh "pip3 install -U ."
          }
        }
    stage('Test rundoc') {
      steps {
            sh "export PATH=$PATH:~/.local/bin; ./tests/all.sh"
          }
        }
  } //stages

  post {
    failure {
      slackSend(
        channel: '#ft-delivery',
        color: 'bad',
        message: "Rundoc failed to build - Logs available at ${BUILD_URL}"
        )
      }
    } //post
} //pipeline
