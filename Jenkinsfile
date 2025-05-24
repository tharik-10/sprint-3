pipeline {
    agent any

    environment {
        GIT_USER_NAME = "tharik-10"
        GIT_USER_EMAIL = "md.tharik@mygurukulam.co"
        COMMIT_MESSAGE = "This is the sample message that shows the commit signoff with using declarative pipeline"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Configure Git') {
            steps {
                sh """
                git config user.name "${GIT_USER_NAME}"
                git config user.email "${GIT_USER_EMAIL}"
                """
            }
        }

        stage('Make Dummy Change') {
            steps {
                sh """
                echo "Pipeline ran on: \$(date)" > pipeline-log.txt
                git add pipeline-log.txt
                """
            }
        }

        stage('Commit with Sign-off') {
            steps {
                sh """
                git commit -m "${COMMIT_MESSAGE}" -s || echo "No changes to commit."
                """
            }
        }

        stage('Print Commit Message') {
            steps {
                script {
                    def message = sh(script: "git log -1 --pretty=format:'%B'", returnStdout: true).trim()
                    echo "📝 Latest Commit Message:\n${message}"
                }
            }
        }

        stage('Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token1', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                    git push https://${USERNAME}:${PASSWORD}@git@github.com:tharik-10/sprint-3.git HEAD:main || echo "Nothing to push."
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully with commit sign-off."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
