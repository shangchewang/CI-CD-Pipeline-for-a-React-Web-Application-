pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials("netlify-personal-access-token")
        NETLIFY_SITE_ID = 'YOUR SITE ID'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Veverita-Engineering/Customer-Portal'
            }
        }
        stage('Build') {
            steps {
                nodejs(nodeJSInstallationName: 'node-lts') {
                    sh 'node --version'
                    sh 'npm --version'
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Parallel Stage') {
            failFast true
            parallel {
                stage('Test') {
                    steps {
                        nodejs(nodeJSInstallationName: 'node-lts') {
                            sh 'npm test -- --reporters=jest-junit'
                        }
                    }
                }
                stage('SCA') {
                    steps {
                        snykSecurity(
                            snykInstallation: 'snyk@latest',
                            snykTokenId: 'snyk-api-token',
                        )
                    }
                }
            }
        }

        stage('Deploy staging') {
            steps {
                nodejs(nodeJSInstallationName: 'node-lts') {
                    sh 'node_modules/.bin/netlify deploy --dir=build'
                }
            }
        }

        stage('Deploy prod') {
            steps {
                nodejs(nodeJSInstallationName: 'node-lts') {
                    echo "Deploying site: ${NETLIFY_SITE_ID}"
                    sh 'node_modules/.bin/netlify status'
                    sh 'node_modules/.bin/netlify deploy --dir=build --prod'
                    sh 'curl https://YOUR-SITE-NAME.netlify.app/ | grep -q "React App"'
                }
            }
        }
    }

    post {
        always {
            junit 'reports/junit.xml'
        }
        success {
            archiveArtifacts artifacts: 'build/**', fingerprint: true
        }
    }
}
