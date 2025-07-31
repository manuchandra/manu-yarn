pipeline {
    agent any
    environment {
        // Define environment variables for JFrog
        JFROG_CLI_HOME = "${env.WORKSPACE}/.jfrog"
        JFROG_URL = 'https://hts2.jfrog.io' // Replace with your Artifactory URL
        JFROG_REPO = 'manu-yarn-npm' // Replace with your Artifactory Python repository
        JFROG_USERNAME = 'manu' // Jenkins credentials ID for username
        JFROG_PASSWORD = 'Password@123' // Jenkins credentials ID for password
        NPM_AUTH_TOKEN = credentials('Jfrog')
    }
    tools {
        jfrog 'jfrog-cli-2.76.1'
        nodejs 'np'
    }
    stages {
        stage('Install dependencies') {
            steps {
                sh 'npm install -g yarn'
                sh 'yarn --version'
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
                    withCredentials([usernamePassword(credentialsId: "Jfrog", usernameVariable: 'JFROG_USERNAME', passwordVariable: 'JFROG_PASSWORD')]) {
                        sh '''
                        git config --global credential.helper '!f() { echo "username=${JFROG_USERNAME}"; echo "password=${JFROG_PASSWORD}"; }; f'
                        git config user.name "Jenkins CI"
                        git checkout main
                        '''
                        // Configure JFrog CLI
                        sh """
                        jf c add --insecure-tls true --url $JFROG_URL --user $JFROG_USERNAME --password $JFROG_PASSWORD --interactive=false
                        jf yarn-config --repo-resolve $JFROG_REPO
                        jf yarn install --build-name=my-yarn-build --build-number=1 
                        jf rt bce my-yarn-build 1
                        jf rt bp my-yarn-build 1
                        """
                        }
                    }
                }
            }
        }
    }
