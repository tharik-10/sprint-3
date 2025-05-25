@Library('my-shared-lib@main') _

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

        stage('Commit Sign-off') {
            steps {
                commitSignoff(
                    gitUser: "${env.GIT_USER_NAME}",
                    gitEmail: "${env.GIT_USER_EMAIL}",
                    commitMessage: "${env.COMMIT_MESSAGE}"
                )
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
