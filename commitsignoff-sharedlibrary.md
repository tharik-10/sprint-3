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
def prepareWorkspace() {
    echo '🧽 Cleaning workspace before checkout...'
    deleteDir()
}

def checkoutCode(String repoUrl, String branch, String credentialsId) {
    withCredentials([usernamePassword(credentialsId: credentialsId, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
        sh """
            git clone https://\$GIT_USERNAME:\$GIT_PASSWORD@${repoUrl} .
            git checkout ${branch}
            git pull origin ${branch} --rebase
        """
    }
}

def configureGit(String userName, String userEmail) {
    sh """
        git config user.name "${userName}"
        git config user.email "${userEmail}"
    """
}

def cleanupWorkspace() {
    echo '🧹 Cleaning up workspace...'
    deleteDir()
}

// This makes the shared library callable without params to run prepare, checkout, configure
def call(Map config) {
    prepareWorkspace()
    checkoutCode(config.repoUrl, config.branch, config.credentialsId)
    configureGit(config.userName, config.userEmail)
}
```
## Use Shared Library in Jenkinsfile

```bash
@Library('my-shared-lib@main') _

pipeline {
    agent any

    environment {
        GIT_USER_NAME = 'tharik-10'
        GIT_USER_EMAIL = 'md.tharik@mygurukulam.co'
        COMMIT_MESSAGE = 'This is the sample message that shows the commit signoff with using declarative pipeline'
    }

    options {
        skipStagesAfterUnstable()
    }

    stages {
        stage('Git Setup') {
            steps {
                gitPipeline(
                    repoUrl: 'github.com/tharik-10/sprint-3.git',
                    branch: 'main',
                    credentialsId: 'github-token1',
                    userName: env.GIT_USER_NAME,
                    userEmail: env.GIT_USER_EMAIL
                )
            }
        }

        stage('Make Dummy Change') {
            steps {
                sh '''
                    echo "Pipeline ran on: $(date)" > pipeline-log.txt
                    git add pipeline-log.txt
                '''
            }
        }

        stage('Commit with Sign-off') {
            steps {
                sh '''
                    git commit -m "${COMMIT_MESSAGE}" -s || echo "No changes to commit."
                '''
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
                    sh '''
                        git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/tharik-10/sprint-3.git
                        git push origin main || echo "Nothing to push."
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                gitPipeline.cleanupWorkspace()
            }
        }

        success {
            echo '✅ Pipeline completed successfully with commit sign-off.'
        }

        failure {
            echo '❌ Pipeline failed.'
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


