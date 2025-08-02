pipeline {
    agent any
    environment {
        JFROG_URL = 'https://hts2.jfrog.io'
        JFROG_REPO = 'manu-yarn-npm'
        JFROG_USERNAME = 'manu'
        NPM_AUTH_TOKEN = credentials('Jfrog')
    }
    tools {
        jfrog 'jfrog-cli-2.78.1'
        nodejs 'np'
    }
    stages {
        stage('Setup Environment and Dependencies') {
            steps {
                script {
                    dir('/var/jenkins_home/workspace/manu-yarn') {
                    
                    sh '''
                        ls -la
                        pwd
                        # Install Yarn v2 (Berry) using npm
                        npm install -g yarnpkg
                        yarn --version # Check Yarn version
                        # Initialize Yarn if needed (this will create a .yarnrc.yml file)
                        # Install project dependencies
                        #yarn install --verbose
                    '''
                    }
                }
            }
        }
        stage('Typescript Build') {
            steps {
                #sh 'yarn build'
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
                            git pull origin main
                            yarn lerna version -y --conventional-commits
                            yarn lerna publish from-git -y
                            jf c add --insecure-tls true --url ''' + JFROG_URL + ''' --user ''' + JFROG_USERNAME + ''' --password ''' + JFROG_PASSWORD + ''' --interactive=false
                            jf yarn-config --repo-resolve ''' + JFROG_REPO + '''
                            jf yarn install --build-name=my-yarn-build --build-number=1
                            jf rt bce my-yarn-build 1
                            jf rt bp my-yarn-build 1
                        '''
                    }
                }
            }
        }
    }
}
