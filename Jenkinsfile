pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_TEST_SITE_ID = credentials('netlify-test-site-id')
        EMAIL_USERNAME = credentials('email-username')
        EMAIL_PASSWORD = credentials('email-password')
        GITHUB_PAT = credentials('github-pat')
        NODE_HOME = 'C:\\Program Files\\nodejs'  // Path to where Node.js is installed
        NPM_BIN = 'C:\\Program Files\\nodejs\\node_modules\\npm\\bin'  // Path to npm binaries
        PATH = "${NODE_HOME}\\;${NPM_BIN}\\;${env.PATH}"  // Add Node.js and npm to PATH
    }

    stages {
        stage('Checkout Repository') {
            steps {
                echo "Checking out repository..."
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/test']], // Ensures 'test' branch is checked out
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'LocalBranch', localBranch: 'test']], // Ensures Jenkins checks out the branch properly
                        userRemoteConfigs: [[
                            url: 'https://github.com/kudaykiranreddy/React_app_AWS.git',
                            credentialsId: 'github-pat'
                        ]]
                    ])
                }
            }
        }

        stage('Debug Branch Name') {
            steps {
                script {
                    // Print the branch to check if the environment variable works correctly
                    def branchName = bat(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    echo "Current branch is: ${branchName}"
                    env.BRANCH_NAME = branchName  // Ensure it's set for later stages
                }
            }
        }

        stage('Set Up Node.js') {
            steps {
                echo "Setting up Node.js environment..."
                script {
                    bat 'node -v'
                    bat 'npm -v'
                }
            }
        }

        stage('Install npx') {
            steps {
                echo "Installing npx..."
                bat 'npm install -g npx'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing dependencies..."
                dir('To_do_app') {
                    bat 'npm ci'
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running tests..."
                dir('To_do_app') {
                    bat 'npm test'
                }
            }
        }

        stage('Build the App') {
            steps {
                echo "Building the application..."
                dir('To_do_app') {
                    bat 'npm run build --verbose'
                }
            }
        }

        stage('List Build Directory') {
            steps {
                echo "Listing build directory..."
                dir('To_do_app') {
                    bat 'dir dist || echo "Build directory not found"'
                }
            }
        }

        stage('Deploy to Netlify (Test)') {
            when {
                expression { 
                    // Checking if the current branch is 'test'
                    return env.GIT_BRANCH == 'origin/test' || env.GIT_BRANCH == 'test'
                }
            }
            steps {
                echo "Deploying to Netlify (Test)..."
                bat '''
                npx netlify deploy ^
                    --auth %NETLIFY_AUTH_TOKEN% ^
                    --site %NETLIFY_TEST_SITE_ID% ^
                    --dir To_do_app\\dist ^
                    --message "Test deployment"
                '''
            }
        }

        stage('Create Pull Request for Test to Prod') {
            when {
                expression { 
                    // Checking if the current branch is 'test'
                    return env.GIT_BRANCH == 'origin/test' || env.GIT_BRANCH == 'test'
                }
            }
            steps {
                echo "Creating pull request from test to prod..."
                bat '''
                echo "%GITHUB_PAT%" | gh auth login --with-token
                gh pr create --base prod --head test --title "Merge Test into Prod" --body "This PR merges changes from the test branch to the prod branch."
                '''
            }
        }

        stage('Send Email Notification') {
            when {
                expression { 
                    // Checking if the current branch is 'test'
                    return env.GIT_BRANCH == 'origin/test' || env.GIT_BRANCH == 'test'
                }
            }
            steps {
                echo "Sending email notification for PR creation..."
                mail to: 'ukalicheti@anergroup.com',
                     subject: 'Pull Request Created: Test to Prod',
                     body: 'A pull request has been created to merge changes from the test branch to the prod branch.',
                     from: 'kudaykiranreddy143@gmail.com',
                     replyTo: 'kudaykiranreddy143@gmail.com'
            }
        }
    }

    post {
        always {
            echo "Cleaning up..."
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
            mail to: 'ukalicheti@anergroup.com',
                 subject: 'Pipeline Failed: Test Deployment',
                 body: 'The pipeline for the test deployment has failed. Please check the logs for more details.',
                 from: 'kudaykiranreddy143@gmail.com',
                 replyTo: 'kudaykiranreddy143@gmail.com'
        }
    }
}
