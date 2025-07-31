
#!/usr/bin/env groovy

@Library('utils@jfrog-test') _

pipeline {
    agent any
    environment {
        // Define environment variables for JFrog
        JFROG_CLI_HOME = "${env.WORKSPACE}/.jfrog"
        JFROG_URL = 'https://hts2.jfrog.io' // Replace with your Artifactory URL
        JFROG_REPO = 'manu-yarn-npm' // Replace with your Artifactory Python repository
        JFROG_USERNAME = 'manu' // Jenkins credentials ID for username
        JFROG_PASSWORD = 'Password@123' // Jenkins credentials ID for password
    }
    tools { 
        jfrog 'jfrog-cli-2.76.1'
        nodejs 'np'
    }

  environment {

    NPM_AUTH_TOKEN = credentials('jfrog-jenkins-integration-eastus')

  }

  stages {

    stage('Install dependencies') {

      steps {

        sh 'yarn install --immutable'

      }

    }

    stage('Typescript build') {

      steps {

        sh 'yarn build'

      }

    }

    stage('Publish to Artifactory') {

      steps {

        script {

          withCredentials([usernamePassword(credentialsId: "githubuser", usernameVariable: '$JFROG_USERNAME', passwordVariable: '$JFROG_PASSWORD')]) {

            sh(

              script: '''

              #!/bin/bash

              git config --global credential.helper '!f() { echo "username=${USERNAME}"; echo "password=${PASSWORD}"; }; f'

              git config user.name "Jenkins CI"

              git checkout npm-yarn-sample

              '''

            )

            jfrog.withNpmCredentials('dev') {

                sh '''

                  yarn lerna version -y --conventional-commits

                  yarn lerna publish from-git -y

                '''

            }

          }

        }

      }

    }

  }

}
