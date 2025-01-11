pipeline {
    agent any

    environment {
        NETFLIFY_SITE_ID = 'something'
        NETFLIFY_AUTH_TOKEN = credentials('auth-token')
    }

    stages {
        stage('Build') {
            agent{
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

        stage('Tests') {
            parallel{ 
                stage('Unit tests') {
                    agent{
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
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
                        node_modules/.bin/serve --version
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }

                    #post {
                    #    always {
                    #        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, KeepAll: false, reportDir: 'public'])
                    #    }
                    #}
                }
            }
        }


        stage('Deploy') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                 npm install netlify-cli
                 node_modules/.bin/netlify --version
                 echo "Deploying to production. Site ID: $NETFLIFY_SITE_ID"
                 node_modules/.bin/netlify status
                 node_modules/.bin/netlify deploy --dir=build --prod
               '''
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
