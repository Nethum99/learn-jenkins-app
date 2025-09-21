pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '33d32978-770a-4595-8827-bec47c1af651'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    
    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    ls -la
                    node  --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la test-results
                '''
            }
        }
        

        stage('Run Tests'){
            parallel{
                stage('Unit Test') {
                     agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
            
                    steps{
                        echo 'Test stage'
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml' 
                        }
                    }
                }

                stage('E2E') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
            
                    /*steps{
                        echo 'Test stage'
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }*/

                    steps{
                        sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 5
                        echo "Running Playwright tests"
                        npx playwright test --reporter=html
                        '''
                    }

                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
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

    post{
      always{
        junit 'jest-results/junit.xml' 
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite HTML Report', reportTitles: '', useWrapperFileDirectly: true])
      }
    }
}