![jenkinsgit2](https://github.com/user-attachments/assets/16efe3a1-64ed-4681-ac2f-e04e907deb24)

# **Jenkins Shared Library for Commit Sign-off (Declarative Pipeline Compatible)**

## Create a Shared Library Repository

* Create a new GitHub repo (e.g., `jenkins-shared-library`).
* Structure:

```
jenkins-shared-library/
├── vars/
│   ├── commitSignoff.groovy
├── resources/
│   └── (optional static templates)
├── src/
│   └── org/myorg/utils/... (if needed we can use it for Class to define)
└── README.md
```
![Screenshot-97 (2)](https://github.com/user-attachments/assets/b6d622e4-efae-4758-a044-c031ebea18d8)

## Create the commitSignoff.groovy File
`vars/commitSignoff.groovy`

```groovy
def call(Map config = [:]) {
    def gitUser = config.gitUser ?: "default-user"
    def gitEmail = config.gitEmail ?: "default@example.com"
    def commitMessage = config.commitMessage ?: "Default commit message"

    echo "🔧 Configuring Git user and email..."
    sh """
    git config user.name "${gitUser}"
    git config user.email "${gitEmail}"
    """

    echo "📄 Making dummy change to pipeline-log.txt..."
    sh """
    echo "Pipeline ran on: \$(date)" > pipeline-log.txt
    git add pipeline-log.txt
    """

    echo "✅ Committing with sign-off..."
    sh """
    git commit -m "${commitMessage}" -s || echo "No changes to commit."
    """

    echo "🚀 Pushing changes to remote..."
    withCredentials([usernamePassword(credentialsId: 'github-token1', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        sh """
        git push https://\${USERNAME}:\${PASSWORD}@github.com/tharik-10/sprint-3.git HEAD:main || echo "Nothing to push."
        """
    }

    echo "📝 Printing latest commit message..."
    def message = sh(script: "git log -1 --pretty=format:'%B'", returnStdout: true).trim()
    echo "📝 Latest Commit Message:\n${message}"
}
```
## Use Shared Library in Jenkinsfile

```bash
@Library('my-shared-lib@main') _

pipeline {
    agent any

    environment {
        GIT_USER_NAME = "tharik-10"
        GIT_USER_EMAIL = "md.tharik@mygurukulam.co"
        COMMIT_MESSAGE = "This is the sample message that shows the Commit Signoff with using Declarative pipeline using Jnekins Shared Library"
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
```
### Configure Shared Library in Jenkins UI
  
![Screenshot-97 (1)](https://github.com/user-attachments/assets/2ad582b8-d2a5-4fc8-87be-8fb3225a745f)

### Run the Pipeline
Now when you run the pipeline, it should:
![Screenshot-100](https://github.com/user-attachments/assets/8c56aa72-02c4-401c-be29-1b6a2357bbf9)

- Checkout the code
![Screenshot-97](https://github.com/user-attachments/assets/0073d8ac-e080-415f-b04c-e17b26a5646c)

![Screenshot-99](https://github.com/user-attachments/assets/78a06677-e14a-44bb-a8e9-ccbb777b389a)


