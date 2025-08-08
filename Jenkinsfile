pipeline {
    agent any
    
    stages {
        /*stage('Build') {
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
        */

        stage('Test') {

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
                npx playwright test
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