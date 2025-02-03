pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_TEST_SITE_ID = credentials('netlify-test-site-id')
        EMAIL_USERNAME = credentials('email-username')
        EMAIL_PASSWORD = credentials('email-password')
        GITHUB_PAT = credentials('github-pat')  // GitHub Personal Access Tokens
        NODE_HOME = 'C:\\Program Files\\nodejs'
        NPM_BIN = 'C:\\Program Files\\nodejs\\node_modules\\npm\\bin'
        PATH = "${NODE_HOME}\\;${NPM_BIN}\\;${env.PATH}"
        GITHUB_REPO = 'kudaykiranreddy/React_app_AWS'  // Replace with your GitHub repository
        SOURCE_BRANCH = 'test'  // Source branch for the PR
        TARGET_BRANCH = 'prod'  // Target branch for the PR
    }

    stages {
        stage('Checkout Repository') {
            steps {
                echo "Checking out repository..."
                script {
                    checkout([ 
                        $class: 'GitSCM', 
                        branches: [[name: '*/test']], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [[$class: 'LocalBranch', localBranch: 'test']], 
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
                    def branchName = bat(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    echo "Current branch is: ${branchName}"
                    env.BRANCH_NAME = branchName
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

        stage('Create Pull Request for Test to Prod') {
            when {
                expression {
                    def branchName = bat(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    echo "Branch name is: ${branchName}"  // Debug branch name
                    return branchName == 'test'
                }
            }
            steps {
                echo "Creating pull request from test to prod..."
                script {
                    sh '''
                        git checkout -b temp-merge-branch
                        # Set GitHub credentials to authenticate
                        git config --global user.email "kudaykiranreddy143@gmail.com"
                        git config --global user.name "kudaykiranreddy"
                        # Update the origin URL with the GitHub token
                        git remote set-url origin https://$GITHUB_PAT@github.com/kudaykiranreddy/React_app_AWS.git
                        git push origin temp-merge-branch

                        PR_RESPONSE=$(curl -X POST -H "Authorization: token $GITHUB_PAT" \
                            -H "Accept: application/vnd.github.v3+json" \
                            https://api.github.com/repos/kudaykiranreddy/React_app_AWS/pulls \
                            -d '{
                                "title": "Auto PR: test to prod",
                                "head": "test",
                                "base": "prod",
                                "body": "Auto-generated pull request for merging test into prod."
                            }')
 
                        echo "✅ Pull request created. Please review and merge manually."
                    '''
                }
            }
        }

        stage('Send Email Notification') {
            when {
                expression {
                    def branchName = bat(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    return branchName == 'test'
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
