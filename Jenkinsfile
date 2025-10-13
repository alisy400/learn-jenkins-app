pipeline {
    agent any
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
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la    
                '''
            }
                //npm run build-- This should be put above within the the three apostrophies, but will take too much time and pipeline works fine without it.
                
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
    }
}
