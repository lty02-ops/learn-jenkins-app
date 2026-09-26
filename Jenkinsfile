pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '2146b623-c855-4553-a607-dcec19b2140b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }
    
    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                }
            }
            steps {
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
                echo 'Small change'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Run tests') {
            parallel {
                stage('Unit tests') {
                    agent {
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy staging') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    environment {
                        CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
                    }

                    steps {
                        sh '''
                            netlify --version
                            echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                            netlify status
                            netlify deploy --dir=build --no-build --json > deploy-output.json
                            CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
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

                    environment {
                        CI_ENVIRONMENT_URL = 'https://cheery-moxie-df21a4.netlify.app'
                    }

                    steps {
                        sh '''
                            node --version
                            netlify --version
                            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                            netlify status
                            netlify deploy --dir=build --prod --no-build
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
        }
    }
}