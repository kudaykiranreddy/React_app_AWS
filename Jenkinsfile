pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_TEST_SITE_ID = credentials('netlify-test-site-id')
        EMAIL_USERNAME = credentials('email-username')
        EMAIL_PASSWORD = credentials('email-password')
        GITHUB_PAT = credentials('github-pat')
        NODE_HOME = 'C:\\Program Files\\nodejs'  // Path to where Node.js is installed
        NPM_BIN = 'C:\\Program Files\\nodejs\\node_modules\\npm\\bin'  // Path to where npm binaries are installed
        PATH = "${NODE_HOME}\\;${NPM_BIN}\\;${env.PATH}"  // Add both Node.js and npm to the PATH
    }

    stages {
        stage('Checkout Repository') {
            steps {
                echo "Checking out repository..."
                checkout scm
            }
        }

        stage('Set Up Node.js') {
            steps {
                echo "Setting up Node.js environment..."
                script {
                    // Verify Node and npm versions
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

        stage('Debug Branch Name') {
            steps {
                script {
                    // Get the current branch name and set the environment variable manually
                    def branchName = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    echo "Current branch is: ${branchName}"
                    env.BRANCH_NAME = branchName
                }
            }
        }

        stage('Deploy to Netlify (Test)') {
            when {
                expression { env.BRANCH_NAME == 'test' }
            }
            steps {
                echo "Deploying to Netlify (Test)..."
                bat '''
                npx netlify deploy ^
                    --auth $NETLIFY_AUTH_TOKEN ^
                    --site $NETLIFY_TEST_SITE_ID ^
                    --dir To_do_app\\dist ^
                    --message "Test deployment"
                '''
            }
        }

        stage('Create Pull Request for Test to Prod') {
            when {
                expression { env.BRANCH_NAME == 'test' }
            }
            steps {
                echo "Creating pull request from test to prod..."
                bat '''
                echo "${GITHUB_PAT}" | gh auth login --with-token
                gh pr create --base prod --head test --title "Merge Test into Prod" --body "This PR merges changes from the test branch to the prod branch."
                '''
            }
        }

        stage('Send Email Notification') {
            when {
                expression { env.BRANCH_NAME == 'test' }
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
