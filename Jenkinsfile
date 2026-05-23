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
                            // FIXED: Replaced 'jest-results/junit.xml' with deep wildcard lookup
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
                withCredentials([string(credentialsId: 'netlify-token', variable: 'NETLIFY_AUTH_TOKEN')]) {
                    sh '''
                        npm install netlify-cli
                        node_modules/.bin/netlify --version
                        echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                        
                        echo "=== Verifying Credentials ==="
                        node_modules/.bin/netlify status
                        
                        echo "=== Uploading Pre-Built Assets to Netlify ==="
                        # التعديل الحاسم: نأمر الأداة برفع الفولدر الجاهز مباشرة ومنع بناء الكود في السحاب
                        node_modules/.bin/netlify deploy --dir=build --prod --site $NETLIFY_SITE_ID --build=false
                    '''
                }
            }
        }
        
                
            
        
    }
}