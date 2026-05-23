pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '38f0ded9-a0c7-41ab-9781-21249a48ae6e'
    }
    
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'CI=true npm test'
                    }
                    post {
                        always {
                            junit '**/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: ''])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }

    steps {
        withCredentials([
            string(
                credentialsId: 'netlify-token',
                variable: 'NETLIFY_AUTH_TOKEN'
            )
        ]) {

            sh '''
                npm install netlify-cli@20.1.1

                echo "=== Netlify CLI Version ==="
                node_modules/.bin/netlify --version

                echo "=== Checking Auth ==="
                node_modules/.bin/netlify status

                echo "=== Deploying ==="
                node_modules/.bin/netlify deploy \
                  --site=$NETLIFY_SITE_ID \
                  --dir=build \
                  --prod \
                  --auth=$NETLIFY_AUTH_TOKEN
            '''
        }
    }
}
    }
}