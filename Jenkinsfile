pipeline {
    agent any

    stages {
        stage('Verify Node.js & npm') {
            steps {
                script {
                    echo "🔍 Checking Node.js and npm versions..."
                    sh '''
                        export PATH=$PATH:/var/lib/jenkins/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/NodeJS_18/bin
                        if ! command -v node &> /dev/null; then echo "❌ Node.js not found! Install it on Jenkins."; exit 1; fi
                        if ! command -v npm &> /dev/null; then echo "❌ npm not found! Install it on Jenkins."; exit 1; fi
                        echo "✅ Node.js Version: $(node -v)"
                        echo "✅ npm Version: $(npm -v)"
                    '''
                }
            }
        }
    }
}
