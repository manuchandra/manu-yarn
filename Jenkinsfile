pipeline {
    agent any
    environment {
        // Define environment variables for JFrog
        JFROG_URL = 'https://hts2.jfrog.io' 
        JFROG_REPO = 'manu-yarn-npm' 
        JFROG_USERNAME = 'manu' 
        // Note: JFROG_PASSWORD should not be in this block for security
        
        // This is a more secure way to handle credentials
        NPM_AUTH_TOKEN = credentials('Jfrog')
    }
    tools {
        jfrog 'jfrog-cli-2.78.1'
        nodejs 'np'
        // We will manage Node.js and Yarn installation directly
        // rather than relying on a pre-configured Jenkins tool.
    }
    
    stages {
        stage('Setup Environment and Dependencies') {
            steps {
                script {
                    // Use a temporary directory to avoid conflicts with global tools
                    sh '''
                        # Install a specific Node.js version if not available
                        # You may need to adjust this command based on your agent OS

                        # Manually delete old Yarn installations and cache to prevent conflicts
                        rm -rf ~/.yarn
                        rm -rf ~/.yarn-cache
                        
                        # Install the latest stable Yarn version correctly
                        npm install -g yarnpkg

                        # Verify that the correct Yarn version is now in use
                        yarn --version
                        
                        # Install project dependencies
                        yarn install
                    '''
                }
            }
        }
        
        stage('Typescript Build') {
            steps {
                // Assuming 'yarn build' is defined in package.json and 'typescript' is a dev dependency
                sh 'yarn build'
            }
        }
        
        stage('Publish to Artifactory') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "Jfrog", usernameVariable: 'JFROG_USERNAME', passwordVariable: 'JFROG_PASSWORD')]) {
                        sh '''
                            # Git configuration for Lerna
                            git config --global credential.helper '!f() { echo "username=${JFROG_USERNAME}"; echo "password=${JFROG_PASSWORD}"; }; f'
                            git config user.name "Jenkins CI"
                            git checkout main
    
                            # Lerna commands
                            yarn lerna version -y --conventional-commits
                            yarn lerna publish from-git -y
    
                            # Secure JFrog CLI configuration
                            jf c add --insecure-tls true --url ''' + JFROG_URL + ''' --user ''' + JFROG_USERNAME + ''' --password ''' + JFROG_PASSWORD + ''' --interactive=false
    
                            # JFrog Yarn commands
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
