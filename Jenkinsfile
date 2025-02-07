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
        SONARQUBE_TOKEN = credentials('sonar_token_id')
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

        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarQubeScanner'  // Ensure this matches the tool name in Jenkins settings
            }
            steps {
                script {
                    echo "🔍 Running SonarQube analysis..."
                    sh '''
                        cd React_app_AWS/To_do_app
                        $SCANNER_HOME/bin/sonar-scanner \
                          -Dsonar.projectKey=my-react-app \
                          -Dsonar.projectName="My React Application" \
                          -Dsonar.projectVersion=1.0.0 \
                          -Dsonar.sources=src \
                          -Dsonar.tests=src/__tests__ \
                          -Dsonar.exclusions=**/node_modules/**,**/*.test.js,**/*.spec.js \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                          -Dsonar.host.url=http://localhost:9001 \
                          -Dsonar.login=$SONARQUBE_TOKEN || { echo "❌ SonarQube analysis failed"; exit 1; }
                    '''
                }
            }
        }

        stage('Deploy to Netlify (Test)') {
            steps {
                script {
                    echo "🚀 Deploying to Netlify (Test environment)..."
                    sh '''
                        cd React_app_AWS/To_do_app
                        npx netlify deploy --auth $NETLIFY_AUTH_TOKEN --site $NETLIFY_SITE_ID --dir=build --prod || { echo "❌ Deployment failed"; exit 1; }
                        echo "✅ Deployment successful!"
                    '''
                }
            }
        }

        stage('Send Email Notification') {
            steps {
                echo "📧 Sending email notification for PR creation..."
                mail to: 'ukalicheti@anergroup.com',
                     subject: 'Pull Request Created: Test to Prod',
                     body: 'A pull request has been created to merge changes from the test branch to the prod branch.',
                     from: 'kudaykiranreddy143@gmail.com',
                     replyTo: 'kudaykiranreddy143@gmail.com'
            }
        }
    }



    post {
        failure {
            mail to: 'ukalicheti@anergroup.com',
                 subject: 'Build Failed: React App Deployment',
                 body: 'The Jenkins build has failed during one of the pipeline stages.',
                 from: 'kudaykiranreddy143@gmail.com',
                 replyTo: 'kudaykiranreddy143@gmail.com'
        }
    }
}
