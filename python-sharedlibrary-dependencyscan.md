# **Jenkins Shared Libary Pipeline for Python Dependency Scanning**

## Define the Pipeline
```groovy
@Library('my-shared-lib@main') _

pipeline {
    agent any

    environment {
        ATTENDANCE_REPO     = 'https://github.com/OT-MICROSERVICES/attendance-api.git'
        NOTIFICATION_REPO   = 'https://github.com/OT-MICROSERVICES/notification-worker.git'
    }

    stages {
        stage('Clone Code') {
            steps {
                script {
                    cloneAndPost(ATTENDANCE_REPO, NOTIFICATION_REPO)
                }
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

    post cloneAndPost.postActions()
}
```
## Jenkins Shared Library Folder Directory and Script
```groovy
// vars/cloneAndPost.groovy
def call(Map config = [:]) {
    pipeline {
        agent any

        environment {
            ATTENDANCE_REPO   = config.attendanceRepo ?: 'https://github.com/OT-MICROSERVICES/attendance-api.git'
            NOTIFICATION_REPO = config.notificationRepo ?: 'https://github.com/OT-MICROSERVICES/notification-worker.git'
        }

        stages {
            stage('Clone Repositories') {
                steps {
                    echo 'Cloning repositories...'
                    sh '''
                        git clone ${ATTENDANCE_REPO}
                        git clone ${NOTIFICATION_REPO}
                    '''
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
            always {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }
    }
}
```
## Click Build Now to trigger the job.

## Monitor the **Console Output** for Dependency Scanning, pip-audit
