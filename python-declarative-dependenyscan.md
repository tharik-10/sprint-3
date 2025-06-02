# **Declarative Jenkins Pipeline for Python Dependency Scanning**

## Define the Pipeline
```groovy
pipeline {
    agent any

    environment {
        ATTENDANCE_REPO = 'https://github.com/OT-MICROSERVICES/attendance-api.git'
        NOTIFICATION_REPO = 'https://github.com/OT-MICROSERVICES/notification-worker.git'
    }

    stages {

        stage('Install Packages') {
            steps {
                echo 'Installing required packages...'
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y python3 python3-pip python3-venv
                '''
            }
        }

        stage('Clone Repositories') {
            steps {
                echo 'Cloning Attendance API and Notification Worker repos...'
                sh '''
                    sudo rm -rf notification-worker attendance-api
                    git clone ${ATTENDANCE_REPO}
                    git clone ${NOTIFICATION_REPO}
                '''
            }
        }

        stage('Audit Attendance API Dependencies') {
            steps {
                dir('attendance-api') {
                    echo 'Auditing Attendance API dependencies...'
                    sh '''
                        python3 -m venv venv1
                        . venv1/bin/activate
                        pip install --upgrade pip safety
                        pip freeze > requirements.txt
                        safety check -r requirements.txt > safety_report_attendance.txt || true
                        deactivate

                        mkdir -p ../html-reports/attendance
                        echo "<h2>Attendance API - Safety Report</h2><pre>$(cat safety_report_attendance.txt)</pre>" > ../html-reports/attendance/index.html
                    '''
                    archiveArtifacts artifacts: 'attendance-api/safety_report_attendance.txt', allowEmptyArchive: true
                }
            }
        }

        stage('Audit Notification Worker Dependencies') {
            steps {
                dir('notification-worker') {
                    echo 'Auditing Notification Worker dependencies...'
                    sh '''
                        python3 -m venv venv2
                        . venv2/bin/activate
                        pip install --upgrade pip safety
                        pip freeze > requirements.txt
                        safety check -r requirements.txt > safety_report_notification.txt || true
                        deactivate

                        mkdir -p ../html-reports/notification
                        echo "<h2>Notification Worker - Safety Report</h2><pre>$(cat safety_report_notification.txt)</pre>" > ../html-reports/notification/index.html
                    '''
                    archiveArtifacts artifacts: 'notification-worker/safety_report_notification.txt', allowEmptyArchive: true
                }
            }
        }

        stage('Publish Safety Reports') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'html-reports/attendance',
                    reportFiles: 'index.html',
                    reportName: 'Safety Report - Attendance API'
                ])
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'html-reports/notification',
                    reportFiles: 'index.html',
                    reportName: 'Safety Report - Notification Worker'
                ])
            }
        }
    }

    post {
        success {
            echo '🎉 Build succeeded!'
        }

        failure {
            echo '❌ Build failed!'
        }

        unstable {
            echo '⚠️ Audit found vulnerabilities.'
        }
    }
}
def notify(status, priority, slackChannel, emailRecipients) {
    def slackColors = [SUCCESS: 'good', FAILURE: 'danger']
    def slackTitles = [
        SUCCESS: "✅ SUCCESS: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER})",
        FAILURE: "❌ FAILURE: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER})"
    ]
    def slackTexts = [
        SUCCESS: """
            *Job Name:* ${env.JOB_NAME}
            *Build No:* ${env.BUILD_NUMBER}
            *Triggered By:* [Started by user admin]
            *Job Done By:* Mohamed Tharik

            This job has completed successfully. 🎉
            *🔗 <${env.BUILD_URL}|Check logs here>*
        """,
        FAILURE: """
            *Job Name:* ${env.JOB_NAME}
            *Build No:* ${env.BUILD_NUMBER}
            *Triggered By:* [Started by user admin]
            *Job Done By:* Mohamed Tharik

            This job has failed. ❗
            *🔗 <${env.BUILD_URL}|Check logs here>*
        """
    ]

    def emailSubjects = [
        SUCCESS: "✅ SUCCESS: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER})",
        FAILURE: "❌ FAILURE: ${env.JOB_NAME} (Build #${env.BUILD_NUMBER})"
    ]

    def emailBodies = [
        SUCCESS: """
            <div style="border: 1px solid #c3e6cb; background-color: #d4edda; padding: 10px; border-radius: 5px;">
                <h2 style="color: #155724;">✅ Jenkins Job SUCCESS</h2>
                <p><b>Job Name:</b> ${env.JOB_NAME}</p>
                <p><b>Build No:</b> ${env.BUILD_NUMBER}</p>
                <p><b>Triggered By:</b> [Started by user admin]</p>
                <p><b>Owner:</b> Mohamed Tharik</p>
                <p>This job has completed successfully. 🎉</p>
                <p>🔗 <a href='${env.BUILD_URL}' style="color: #155724;">Check logs here</a></p>
            </div>
        """,
        FAILURE: """
            <div style="border: 1px solid #f5c6cb; background-color: #f8d7da; padding: 10px; border-radius: 5px;">
                <h2 style="color: #721c24;">❌ Jenkins Job FAILED</h2>
                <p><b>Job Name:</b> ${env.JOB_NAME}</p>
                <p><b>Build No:</b> ${env.BUILD_NUMBER}</p>
                <p><b>Triggered By:</b> [Started by user admin]</p>
                <p><b>Owner:</b> Mohamed Tharik</p>
                <p>This job has failed. ❗</p>
                <p>🔗 <a href='${env.BUILD_URL}' style="color: #721c24;">Check logs here</a></p>
            </div>
        """
    ]

    slackSend(
        channel: slackChannel,
        attachments: [[
            color: slackColors[status],
            title: slackTitles[status],
            text: slackTexts[status],
            mrkdwn_in: ["text"]
        ]]
    )

    emailext(
        subject: emailSubjects[status],
        body: emailBodies[status],
        to: emailRecipients,
        mimeType: 'text/html',
        recipientProviders: [[$class: 'RequesterRecipientProvider']]
    )
}

```
## Click **Build Now** to trigger the job.

## Monitor the **Console Output** for Dependency Scanning, pip-audit

