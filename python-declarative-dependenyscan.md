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
```
## Click **Build Now** to trigger the job.

## Monitor the **Console Output** for Dependency Scanning, pip-audit

