pipeline {
    agent any
    environment {
        // Define environment variables for JFrog
        JFROG_CLI_HOME = "${env.WORKSPACE}/.jfrog"
        JFROG_URL = 'https://hts2.jfrog.io' // Replace with your Artifactory URL
        JFROG_REPO = 'manu-yarn-npm' // Replace with your Artifactory Python repository
        JFROG_USERNAME = 'manu' // Jenkins credentials ID for username
        JFROG_PASSWORD = 'Password@123' // Jenkins credentials ID for password
        NPM_AUTH_TOKEN = credentials('jfrog-jenkins-integration-eastus')
    }
    tools {
        jfrog 'jfrog-cli-2.76.1'
        nodejs 'np'
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
                    withCredentials([usernamePassword(credentialsId: "githubuser", usernameVariable: 'JFROG_USERNAME', passwordVariable: 'JFROG_PASSWORD')]) {
                        sh '''
                        git config --global credential.helper '!f() { echo "username=${JFROG_USERNAME}"; echo "password=${JFROG_PASSWORD}"; }; f'
                        git config user.name "Jenkins CI"
                        git checkout npm-yarn-sample
                        '''
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
