pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '4ad45eed-8ba6-4de3-a95d-9e11c57a8776'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                npx serve -s build &
                sleep 10
                npx playwright test --reporter=html
                '''
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
                npm install netlify-cli node-jq
                npx netlify deploy --dir=build --json > deploy-output.json
                '''
                script {
                    env.STAGING_URL = sh(
                        script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Staging E2E') {
            agent {
                docker {
                    image 'my-playwright'
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
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Yes'
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
                npm install netlify-cli
                npx netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'my-playwright'
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
        }
    }
}