pipeline {
    agent any

    stages {
        // 1. مرحلة البناء (شغالة كفاءة)
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

        // 2. مرحلة الاختبارات المتوازية
        stage('Tests') {
            parallel {
                
                // الفرع الأول: الـ Unit tests (تم تصليحه)
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        // تفعيل CI=true إجباري عشان التيست يقفل وميعلقش
                        sh 'CI=true npm test'
                    }
                    post {
                        always {
                            // التعديل الذهبي: البحث عن ملف الـ XML في أي مكان في المشروع تلقائياً
                            junit '**/junit.xml'
                        }
                    }
                }

                // الفرع الثاني: الـ E2E (شغال كفاءة 100%)
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

        // 3. مرحلة الـ Deploy (هتشتغل لأول مرة بعد نجاح التيست)
        stage('Deploy') {
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
                '''
            }
        }
    }
}