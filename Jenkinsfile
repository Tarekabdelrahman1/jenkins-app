pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '03d4042d-476c-4668-9ce8-34352dad73e4'
    }

    stages {

        // 1. مرحلة البناء (طبخ الكود وتوليد مجلد build)
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        // 2. مرحلة النشر عل طول (تخطينا التيست)
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                withCredentials([string(credentialsId: 'netlify-token', variable: 'NETLIFY_AUTH_TOKEN')]) {
                    sh '''
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                        
                        echo "=== Verifying Credentials ==="
                        node_modules/.bin/netlify status
                        
                        echo "=== Uploading Build to Netlify ==="
                        node_modules/.bin/netlify deploy --dir=build --prod
                    '''
                }
            }
        }
    }
}