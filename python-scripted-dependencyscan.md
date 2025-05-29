# **Scripted Jenkins Pipeline for Python Dependency Scanning**

## Define the Pipeline
```groovy
node {
    // Define environment variables as Groovy variables
    def ATTENDANCE_REPO = 'https://github.com/OT-MICROSERVICES/attendance-api.git'
    def NOTIFICATION_REPO = 'https://github.com/OT-MICROSERVICES/notification-worker.git'

    try {
        stage('Clone Repositories') {
            echo 'Cloning repositories...'
            sh """
                git clone ${ATTENDANCE_REPO}
                git clone ${NOTIFICATION_REPO}
            """
        }

        stage('Audit Attendance API') {
            dir('attendance-api') {
                sh '''
                    python3 -m venv venv
                    bash -c "source venv/bin/activate && \
                    pip install pip-audit && \
                    pip freeze > requirements.txt && \
                    pip-audit -r requirements.txt > audit_attendance.txt || true && \
                    deactivate"
                '''
                def found = sh(script: 'grep -q "Vulnerability:" audit_attendance.txt', returnStatus: true)
                if (found == 0) {
                    currentBuild.result = 'UNSTABLE'
                    echo 'Vulnerabilities found in Attendance API.'
                }
                archiveArtifacts 'audit_attendance.txt'
            }
        }

        stage('Audit Notification Worker') {
            dir('notification-worker') {
                sh '''
                    python3 -m venv venv
                    bash -c "source venv/bin/activate && \
                    pip install pip-audit && \
                    pip freeze > requirements.txt && \
                    pip-audit -r requirements.txt > audit_notification.txt || true && \
                    deactivate"
                '''
                def found = sh(script: 'grep -q "Vulnerability:" audit_notification.txt', returnStatus: true)
                if (found == 0) {
                    currentBuild.result = 'UNSTABLE'
                    echo 'Vulnerabilities found in Notification Worker.'
                }
                archiveArtifacts 'audit_notification.txt'
            }
        }

        if (currentBuild.result == null) {
            currentBuild.result = 'SUCCESS'
        }
    } catch (err) {
        currentBuild.result = 'FAILURE'
        echo "Build failed due to: ${err}"
        throw err
    } finally {
        if (currentBuild.result == 'SUCCESS') {
            echo '🎉 Build succeeded!'
        } else if (currentBuild.result == 'UNSTABLE') {
            echo '⚠️ Audit found vulnerabilities.'
        } else if (currentBuild.result == 'FAILURE') {
            echo '❌ Build failed!'
        }
        // Clean up workspace
        echo 'Cleaning up workspace...'
        deleteDir()
    }
}
```
## Click **Build Now** to trigger the job.

## Monitor the **Console Output** for Dependency Scanning, pip-audit
