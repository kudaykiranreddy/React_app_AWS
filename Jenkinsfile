pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_TEST_SITE_ID = credentials('netlify-test-site-id')
        EMAIL_USERNAME = credentials('email-username')
        EMAIL_PASSWORD = credentials('email-password')
        GITHUB_PAT = credentials('github-pat')
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
                tool name: 'NodeJS', type: 'NodeJSInstallation'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing dependencies..."
                dir('To_do_app') {
                    sh 'npm ci'
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running tests..."
                dir('To_do_app') {
                    sh 'npm test'
                }
            }
        }

        stage('Build the App') {
            steps {
                echo "Building the application..."
                dir('To_do_app') {
                    sh 'npm run build --verbose'
                }
            }
        }

        stage('List Build Directory') {
            steps {
                echo "Listing build directory..."
                dir('To_do_app') {
                    sh 'ls -alh dist || echo "Build directory not found"'
                }
            }
        }

        stage('Deploy to Netlify (Test)') {
            when {
                expression { env.BRANCH_NAME == 'test' }
            }
            steps {
                echo "Deploying to Netlify (Test)..."
                sh '''
                npx netlify deploy \
                    --auth $NETLIFY_AUTH_TOKEN \
                    --site $NETLIFY_TEST_SITE_ID \
                    --dir To_do_app/dist \
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
                sh '''
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
