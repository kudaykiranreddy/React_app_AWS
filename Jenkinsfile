pipeline {
    agent any

    tools {
        nodejs "NodeJS_18"  // Ensure this matches the tool name in Jenkins settings
    }

    stages {
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
    }
}
