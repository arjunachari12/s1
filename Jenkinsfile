pipeline {
    agent any

    environment {
        BRANCH_NAME = "${env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'unknown'}"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')  // Set pipeline timeout
        buildDiscarder(logRotator(numToKeepStr: '10')) // Keep last 10 builds
    }

    triggers {
        pollSCM('H/5 * * * *') // Check for changes every 5 minutes
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "Checking out branch: ${BRANCH_NAME}"
                    if (env.BRANCH_NAME) {
                        checkout scm
                    } else {
                        sh "git clone -b main https://github.com/your-repo.git ."
                    }
                }
            }
        }

        stage('Branch-Based Execution') {
            steps {
                script {
                    def IS_PR = env.CHANGE_ID ? true : false
                    if (IS_PR) {
                        echo "Executing for Pull Request #${env.CHANGE_ID}"
                    } else if (BRANCH_NAME == 'main') {
                        echo "Executing for main branch (Production Build)"
                    } else if (BRANCH_NAME == 'develop') {
                        echo "Executing for develop branch (Staging Build)"
                    } else if (BRANCH_NAME.startsWith('feature/')) {
                        echo "Executing for Feature Branch: ${BRANCH_NAME}"
                    } else {
                        echo "Executing for other branches"
                    }
                }
            }
        }

        stage('Parallel Execution') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        script {
                            echo "Running unit tests..."
                            sh "echo 'Test execution in progress...'"
                        }
                    }
                }

                stage('Linting') {
                    steps {
                        script {
                            echo "Running code linting..."
                            sh "echo 'Lint check in progress...'"
                        }
                    }
                }
            }
        }

        stage('Build and Deploy') {
            steps {
                script {
                    def IS_PR = env.CHANGE_ID ? true : false
                    if (!IS_PR) {  // Only deploy if it's NOT a PR
                        echo "Building the project..."
                        sh "echo 'Build process running...'"
                        
                        if (BRANCH_NAME == 'main') {
                            echo "Deploying to Production..."
                        } else if (BRANCH_NAME == 'develop') {
                            echo "Deploying to Staging..."
                        } else {
                            echo "Deploying to Test Environment..."
                        }
                    } else {
                        echo "Skipping deployment for PR"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed!"
        }
        success {
            echo "✅ Build succeeded!"
        }
        failure {
            echo "❌ Build failed!"
        }
    }
}
