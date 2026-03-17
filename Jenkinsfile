pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '4ad45eed-8ba6-4de3-a95d-9e11c57a8776'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
         stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
                    
                }
            }
            steps{
                sh '''
                aws --version
                '''
            }
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
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm ci

                echo "Deploying to staging..."

                npx netlify deploy \
                  --dir=build \
                  --json \
                  --auth=$NETLIFY_AUTH_TOKEN \
                  --site=$NETLIFY_SITE_ID > deploy-output.json

                cat deploy-output.json
                '''

                script {
                    env.STAGING_URL = sh(
                        script: "cat deploy-output.json | jq -r '.deploy_url'",
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
                echo "Testing staging URL: $CI_ENVIRONMENT_URL"
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
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm ci

                echo "Deploying to production..."

                npx netlify deploy \
                  --dir=build \
                  --prod \
                  --auth=$NETLIFY_AUTH_TOKEN \
                  --site=$NETLIFY_SITE_ID
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
                echo "Testing production URL: $CI_ENVIRONMENT_URL"
                npm ci
                npx playwright test --reporter=html
                '''
            }
        }
    }
}