pipeline {
    agent any
 
    tools {
        nodejs "NodeJS_18"
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
 
        stage('Verify Node.js & npm') {
            steps {
                script {
                    echo "🔍 Checking Node.js and npm versions..."
                    sh '''
                        if ! command -v node &> /dev/null; then echo "❌ Node.js not found! Install it on Jenkins."; exit 1; fi
                        if ! command -v npm &> /dev/null; then echo "❌ npm not found! Install it on Jenkins."; exit 1; fi
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
                        cd React_app_AWS
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
                        cd React_app_AWS
                        npm test || { echo "❌ Tests failed"; exit 1; }
                        echo "✅ All tests passed!"
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
    }
 
    post {
        success {
            echo "🎉 ✅ Pull request created successfully!"
        }
        failure {
            echo "❌ Failed to create pull request! Check logs for details."
        }
    }
}
