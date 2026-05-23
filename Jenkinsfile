pipeline {
    agent any

    stages {

        stage('Netlify Auth Test') {
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

                        echo "=== Netlify Version ==="
                        node_modules/.bin/netlify --version

                        echo "=== Testing Token ==="
                        node_modules/.bin/netlify sites:list \
                          --auth=$NETLIFY_AUTH_TOKEN
                    '''
                }
            }
        }

    }
}