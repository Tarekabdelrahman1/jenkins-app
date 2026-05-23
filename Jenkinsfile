pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '38f0ded9-a0c7-41ab-9781-21249a48ae6e'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {


        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                '''
            }
        }
		stage('Approval') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    input message: 'Ready to deploy?', ok: 'Yes, I am sure I want to deploy!'
                }
            }
        }
        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        
    }
}
