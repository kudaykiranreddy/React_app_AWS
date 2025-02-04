pipeline {
    agent any

    tools {
        nodejs "NodeJS_18"  // Ensure this matches the tool name in Jenkins settings
    }

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
        NETLIFY_SITE_ID = '0773b2bc-94cd-4bd1-941c-2aebdf8fa106'
        GITHUB_TOKEN = credentials('github_token')
        REPO_URL = "https://github.com/kudaykiranreddy/React_app_AWS.git"
        MAIN_BRANCH = "test"
        PROD_BRANCH = "prod"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "🔄 Checking out code from GitHub..."
                    sh '''
                        rm -rf React_app_AWS || true
                        git clone -b $MAIN_BRANCH $REPO_URL React_app_AWS || { echo "❌ Git clone failed"; exit 1; }
                        echo "✅ Code checkout complete."
                    '''
                }
            }
        }

        stage('Verify Directory Structure') {
            steps {
                script {
                    sh '''
                        echo "📁 Checking if React_app_AWS/To_do_app exists..."
                        ls -l React_app_AWS || { echo "❌ React_app_AWS repo not found!"; exit 1; }
                        ls -l React_app_AWS/To_do_app || { echo "❌ To_do_app directory not found!"; exit 1; }
                        echo "✅ Directory structure verified."
                    '''
                }
            }
        }

        stage('Verify Node.js & npm') {
            steps {
                script {
                    echo "🔍 Checking Node.js and npm versions..."
                    sh '''
                        echo "Using NodeJS from Jenkins tool config..."
                        which node || { echo "❌ Node.js not found!"; exit 1; }
                        which npm || { echo "❌ npm not found!"; exit 1; }
                        echo "✅ Node.js Version: $(node -v)"
                        echo "✅ npm Version: $(npm -v)"
                    '''
                }
            }
        }

        stage('Clean & Install Dependencies') {
            steps {
                script {
                    sh '''
                        echo "🧹 Cleaning old dependencies..."
                        cd React_app_AWS/To_do_app
                        rm -rf node_modules package-lock.json
                        
                        echo "📦 Installing dependencies..."
                        npm install || { echo "❌ Failed to install dependencies"; exit 1; }
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                        echo "🧪 Running test cases..."
                        cd React_app_AWS/To_do_app
                        npm test || { echo "❌ Tests failed"; exit 1; }
                        echo "✅ All tests passed!"
                    '''
                }
            }
        }

        stage('Deploy to Netlify (Test)') {
            steps {
                script {
                    echo "🚀 Deploying to Netlify (Test)..."
                    sh '''
                        cd React_app_AWS/To_do_app  # Ensure you're in the correct directory for deployment
                        git checkout test
                        git pull origin test
                        
                        # Install dependencies and build the project
                        npm install
                        npm run build  # This generates the dist directory (or build folder depending on configuration)

                        # Install Netlify CLI
                        npm install -g netlify-cli

                        # Deploy to Netlify using the correct directory
                        npx netlify deploy --auth $NETLIFY_AUTH_TOKEN --site $NETLIFY_SITE_ID --dir dist --message "Test deployment" || { echo "❌ Test deployment to Netlify failed"; exit 1; }

                        echo "✅ Test deployment successful!"
                    '''
                }
            }
        }

        stage('Create Pull Request for Production Merge') {
            steps {
                script {
                    echo "📌 Creating pull request for merging test into prod..."
                    sh '''
                        cd React_app_AWS
                        git checkout -b temp-merge-branch
                        git config --global user.email "kudaykiranreddy143@gmail.com"
                        git config --global user.name "kudaykiranreddy"
                        git remote set-url origin https://$GITHUB_TOKEN@github.com/kudaykiranreddy/React_app_AWS.git
                        git push origin temp-merge-branch

                        PR_RESPONSE=$(curl -X POST -H "Authorization: token $GITHUB_TOKEN" \
                            -H "Accept: application/vnd.github.v3+json" \
                            https://api.github.com/repos/kudaykiranreddy/React_app_AWS/pulls \
                            -d '{
                                "title": "Merge test into prod",
                                "head": "temp-merge-branch",
                                "base": "prod",
                                "body": "Auto-generated pull request for merging test into prod."
                            }')

                        echo "✅ Pull request created. Please review and merge manually."
                    '''
                }
            }
        }

        stage('Send Email Notification') {
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
            echo "🎉 ✅ Pull request created and deployment successful!"
        }
        failure {
            echo "❌ Pipeline failed!"
            mail to: 'ukalicheti@anergroup.com',
                 subject: 'Pipeline Failed: Test Deployment',
                 body: 'The pipeline for the test deployment has failed. Please check the logs for more details.',
                 from: 'kudaykiranreddy143@gmail.com',
                 replyTo: 'kudaykiranreddy143@gmail.com'
        }
    }
}
