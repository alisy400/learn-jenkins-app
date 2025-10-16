pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '4d08ec4f-cdca-40d8-b2d6-cf30dd133bd6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        
        stage('Build') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps{
                sh  '''
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
                stage('unit test')  {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps   {
                        sh '''
                            mkdir -p jest-results
                            JEST_JUNIT_OUTPUT=jest-results/junit.xml npm test

                        '''
                    }
                    
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                                
                        }
                    }
                }

                stage('E2E')  { 
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps   {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html

                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
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

            steps{
                sh  '''
                npm install netlify-cli@20.1.1
                node_modules/.bin/netlify --version
                echo "Deploying to production. Site_id: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify login                

                '''

            }   
        }
    }
}

