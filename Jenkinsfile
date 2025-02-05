pipeline {
    agent any

    environment {
        SECRET_TOKEN = '116671069735682f9eb990efd6d8e25bc4'  // Secret Token you provided
        BASE_URL = " https://1ae5-115-99-175-167.ngrok-free.app"  // Your Ngrok URL
        APPROVAL_URL = "${BASE_URL}/approve?token=${SECRET_TOKEN}&action=approve"  // Approve URL
        REJECT_URL = "${BASE_URL}/reject?token=${SECRET_TOKEN}&action=reject"  // Reject URL
    }

    parameters {
        string(name: 'ACTION', defaultValue: '', description: 'Action for the pipeline (approve/reject)')
    }

    stages {
        stage('Send Approval Email') {
            steps {
                script {
                    def mailSubject = 'Pipeline Approval Request'
                    def mailBody = """
                        <p>Please approve or reject the pipeline execution:</p>
                        <a href="${APPROVAL_URL}" style="padding: 10px; background-color: green; color: white; text-decoration: none;">Approve</a>
                        <a href="${REJECT_URL}" style="padding: 10px; background-color: red; color: white; text-decoration: none;">Reject</a>
                    """
                    emailext(
                        to: 'yaswanth@middlewaretalents.com',
                        subject: mailSubject,
                        body: mailBody,
                        mimeType: 'text/html'
                    )
                }
            }
        }

        stage('Wait for Approval') {
            steps {
                script {
                    // Wait for 5 minutes (300 seconds) for the user to click either approve or reject
                    timeout(time: 5, unit: 'MINUTES') {
                        waitUntil {
                            // Check if the action parameter is 'approve' or 'reject'
                            return params.ACTION == 'approve' || params.ACTION == 'reject'
                        }
                    }

                    // If no action or rejection, fail the pipeline
                    if (params.ACTION == 'approve') {
                        echo 'Pipeline approved!'
                    } else if (params.ACTION == 'reject' || params.ACTION == '') {
                        echo 'Pipeline rejected or no response received. Marking as failure.'
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }

        stage('Execute Pipeline') {
            when {
                expression { currentBuild.result != 'FAILURE' }
            }
            steps {
                script {
                    echo 'Executing pipeline steps...'
                    // Add your actual build or deployment steps here
                }
            }
        }

        stage('Send Success Email') {
            when {
                expression { currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    emailext(
                        to: 'yaswanth@middlewaretalents.com',
                        subject: 'Pipeline Execution Successful',
                        body: 'The pipeline has completed successfully!',
                        mimeType: 'text/html'
                    )
                }
            }
        }

        stage('Send Failure Email') {
            when {
                expression { currentBuild.result == 'FAILURE' }
            }
            steps {
                script {
                    emailext(
                        to: 'yaswanth@middlewaretalents.com',
                        subject: 'Pipeline Execution Failed',
                        body: 'The pipeline execution failed or was rejected.',
                        mimeType: 'text/html'
                    )
                }
            }
        }
    }
}
