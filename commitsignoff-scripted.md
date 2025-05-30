![jenkinsgit2](https://github.com/user-attachments/assets/16efe3a1-64ed-4681-ac2f-e04e907deb24)

# **Scripted Jenkins Pipeline for Commit Sign-off**

## Create a New Jenkins Pipeline Job

![Screenshot-97](https://github.com/user-attachments/assets/0b261f4e-01cb-4e28-8cf1-3daba5c1fce4)

## Create Jenkinsfile in Root Directory of Repository Using Scripted Pipeline
```bash
node {
    def GIT_USER_NAME = "tharik-10"
    def GIT_USER_EMAIL = "md.tharik@mygurukulam.co"
    def COMMIT_MESSAGE = "This is the sample message that shows the commit signoff with using scripted pipeline"

    try {
        stage('Prepare Workspace') {
            echo "🧽 Cleaning workspace before checkout..."
            deleteDir() // Ensures full cleanup including hidden files
        }

        stage('Checkout Code') {
            withCredentials([usernamePassword(credentialsId: 'github-token1', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                sh """
                    git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tharik-10/sprint-3.git .
                    git checkout main
                    git pull origin main --rebase
                """
            }
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
                sh """
                    git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/tharik-10/sprint-3.git
                    git push origin main || echo "Nothing to push."
                """
            }
        }

        echo "✅ Pipeline completed successfully with commit sign-off."

    } finally {
        stage('Cleanup Workspace') {
            echo "🧹 Cleaning up workspace..."
            deleteDir()
        }
    }
}
```
### Run the Pipeline
- Click **Build Now** on your Jenkins job.

![script-commitsignoff4](https://github.com/user-attachments/assets/f7b27fdb-79df-4239-a780-5c760de0f899)

- Monitor the console output to see:
  - The commit message with sign-off.

![script-commitsignoff3](https://github.com/user-attachments/assets/7a223dc3-1aed-4ed2-bb05-759bd42a1340)

  - Confirmation that the push succeeded or a message indicating there's nothing to push.


