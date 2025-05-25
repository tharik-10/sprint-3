node {
    def GIT_USER_NAME = "tharik-10"
    def GIT_USER_EMAIL = "md.tharik@mygurukulam.co"
    def COMMIT_MESSAGE = "This is the sample message that shows the commit sign-off using scripted pipeline"

    stage('Checkout Code') {
        checkout scm
    }

    stage('Configure Git') {
        sh """
            git config user.name "${GIT_USER_NAME}"
            git config user.email "${GIT_USER_EMAIL}"
        """
    }

    stage('Make Dummy Change') {
        sh """
            echo "Pipeline ran on: \$(date)" > pipeline-log.txt
            git add pipeline-log.txt
        """
    }

    stage('Commit with Sign-off') {
        sh """
            git commit -m "${COMMIT_MESSAGE}" -s || echo "No changes to commit."
        """
    }

    stage('Print Commit Message') {
        def message = sh(script: "git log -1 --pretty=format:'%B'", returnStdout: true).trim()
        echo "📝 Latest Commit Message:\n${message}"
    }

    stage('Push Changes') {
        withCredentials([usernamePassword(credentialsId: 'github-token1', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh '''
                git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/tharik-10/sprint-3.git
                git push origin HEAD:main || echo "Nothing to push."
            '''
        }
    }

    echo "✅ Pipeline completed successfully with commit sign-off."
}
