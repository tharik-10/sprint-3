# **Declarative Jenkins Pipeline for Python Dependency Scanning**

## Define the Pipeline
```groovy
pipeline {
    agent any

    environment {
        ATTENDANCE_REPO     = 'https://github.com/OT-MICROSERVICES/attendance-api.git'
        NOTIFICATION_REPO   = 'https://github.com/OT-MICROSERVICES/notification-worker.git'
    }

    stages {
        stage('Clone Repositories') {
            steps {
                echo 'Cloning repositories...'
                sh '''
                    rm -rf attendance-api notification-worker
                    git clone ${ATTENDANCE_REPO}
                    git clone ${NOTIFICATION_REPO}
                '''
            }
        }

        stage('Audit Attendance API') {
            steps {
                dir('attendance-api') {
                    sh '''
                        python3 -m venv venv
                        bash -c "source venv/bin/activate && \
                        pip install pip-audit && \
                        pip freeze > requirements.txt && \
                        pip-audit -r requirements.txt > audit_attendance.txt || true && \
                        deactivate"
                    '''
                    script {
                        def found = sh(script: 'grep -q "Vulnerability:" audit_attendance.txt', returnStatus: true)
                        if (found == 0) {
                            currentBuild.result = 'UNSTABLE'
                            echo 'Vulnerabilities found in Attendance API.'
                        }
                        archiveArtifacts 'audit_attendance.txt'
                    }
                }
            }
        }

        stage('Audit Notification Worker') {
            steps {
                dir('notification-worker') {
                    sh '''
                        python3 -m venv venv
                        bash -c "source venv/bin/activate && \
                        pip install pip-audit && \
                        pip freeze > requirements.txt && \
                        pip-audit -r requirements.txt > audit_notification.txt || true && \
                        deactivate"
                    '''
                    script {
                        def found = sh(script: 'grep -q "Vulnerability:" audit_notification.txt', returnStatus: true)
                        if (found == 0) {
                            currentBuild.result = 'UNSTABLE'
                            echo 'Vulnerabilities found in Notification Worker.'
                        }
                        archiveArtifacts 'audit_notification.txt'
                    }
                }
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

