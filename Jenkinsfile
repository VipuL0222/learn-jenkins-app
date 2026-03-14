pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '4ad45eed-8ba6-4de3-a95d-9e11c57a8776'
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
                echo 'Small change'
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                '''
            }
        }

        stage('Unit Test') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
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
            steps{
                sh '''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    junit 'jest-results/*.xml'

                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Local Report'
                    ])
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

        stage('Prod E2E') {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{
                NETLIFY_SITE_ID = '4ad45eed-8ba6-4de3-a95d-9e11c57a8776'
                NETLIFY_AUTH_TOKEN = credentials('netlify-token')
                CI_ENVIRONMENT_URL='https://wonderful-dieffenbachia-81a823.netlify.app'
            }

            steps{
                sh '''
                npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    junit 'jest-results/*.xml'

                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright E2E Report'
                    ])
                }
            }
        }

    }
}