pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '4ad45eed-8ba6-4de3-a95d-9e11c57a8776'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }



        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
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
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm ci
                test -f build/index.html
                npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm ci
                
                 serve -s build &
                sleep 10
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    junit 'jest-results/*.xml'

                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Local Report'
                    ])
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm ci
                
                npx netlify --version
                echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                npx netlify deploy --dir=build --json > deploy-output.json
                '''
                script {
                    env.STAGING_URL = sh(
                        script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",
                        returnStdout: true
                    ).trim()
                    echo "Staging URL: ${env.STAGING_URL}"
                }
            }
        }

        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                sh '''
                npm ci
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    junit 'jest-results/*.xml'

                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Staging E2E Report'
                    ])
                }
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do You wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm ci
                
                netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                 netlify status
                npx netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://wonderful-dieffenbachia-81a823.netlify.app'
            }
            steps {
                sh '''
                npm ci
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    junit 'jest-results/*.xml'

                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Prod E2E Report'
                    ])
                }
            }
        }
    }
}