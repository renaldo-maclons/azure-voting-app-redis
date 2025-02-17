pipeline {
    agent {
        label 'podman'
    }
    stages {
        stage('Pre-Build notification') {
            steps {
                // Send a notification to Slack no matter the build result
                slackSend (color: '#0000ff', message: "Starting the build for the job: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            }
        }
        stage('Verify Branch') {
            steps {
                echo "$GIT_BRANCH"
            }
        }
        stage('Docker Build') {
            steps {
                sh(script: 'podman-compose build')
            }
        }
        stage('Start App') {
            steps {
                sh(script: 'podman-compose up -d')
            }
        }
        stage('Run Tests') {
            steps {
                sh(script: 'pytest ./tests/test_sample.py')
            }
            post {
                success {
                    echo "Testing Passed! :)"
                }
                failure {
                    echo "Testing Failed! :("
                }
            }
        }
        stage('Docker Push') {
            steps {
                echo "Running in $WORKSPACE"
                dir("$WORKSPACE/azure-vote") {
                    script {
                        docker.withRegistry('', 'dockerhub') {
                            def image = docker.build('renaldomaclons/bmcicdpwj:2024')
                            image.push()
                        }
                    }
                }
            }
            post {
                success {
                    echo "Docker image pushed to registry! :)"
                }
                failure {
                    echo "Docker image upload failed! :("
                }
            }
        }
    }
    post {
        always {
            // Send a notification to Slack no matter the build result
            slackSend (color: '#FFFF00', message: "Build Finished: ${env.JOB_NAME} #${env.BUILD_NUMBER} - Status: ${currentBuild.currentResult}")
            sh(script: 'podman-compose down')
        }
        success {
            // If build was successful, send a green message
            slackSend (color: '#00FF00', message: "Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            // If build failed, send a red message
            slackSend (color: '#FF0000', message: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}. Check console output: ${env.BUILD_URL}")
        }
        unstable {
            // If build was unstable, send a yellow message
            slackSend (color: '#FFFF00', message: "Unstable: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        aborted {
            // If build was aborted, send a grey message
            slackSend (color: '#CCCCCC', message: "Aborted: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}