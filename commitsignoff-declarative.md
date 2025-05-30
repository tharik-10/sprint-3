![jenkinsgit2](https://github.com/user-attachments/assets/16efe3a1-64ed-4681-ac2f-e04e907deb24)

# **Declarative Jenkins Pipeline for Commit Sign-off**


## Create a new Jenkins Pipeline job


![commitsignoff-declarative2](https://github.com/user-attachments/assets/2056287e-da36-4ece-96bf-d04b520cd0a0)

## What the Pipeline Does

![commitsign-declarative3](https://github.com/user-attachments/assets/0f0e6b2d-21ae-4ab1-9ff2-8e96c43db665)
![commitsignoff-declarative4](https://github.com/user-attachments/assets/f82997b9-d394-4c83-b792-ed66521537f1)

## Create Jenkinsfile in Root Directory of Repository Using Declarative Pipeline
```bash
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
        stage('Prepare Workspace') {
            steps {
                echo '🧽 Cleaning workspace before checkout...'
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token1', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/tharik-10/sprint-3.git .
                        git checkout main
                        git pull origin main --rebase
                    '''
                }
            }
        }

        stage('Configure Git') {
            steps {
                sh '''
                    git config user.name "${GIT_USER_NAME}"
                    git config user.email "${GIT_USER_EMAIL}"
                '''
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
            echo '🧹 Cleaning up workspace...'
            deleteDir()
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
### Run the Pipeline
- Click **Build Now** on your Jenkins job.

![commitsignoff-declarative](https://github.com/user-attachments/assets/441bfd16-48ed-4902-b3d4-6930cca96a68)

- Watch the console output to see:
  - Commit message with sign-off.
  
![commitsignoff-declarative5](https://github.com/user-attachments/assets/2e144788-80e1-4db0-b9b2-b5e994663c50)

  - Confirmation that push succeeded or if nothing to push.

![commitsignoff-declarative6](https://github.com/user-attachments/assets/32a23551-b62a-431e-bb9f-f7e4010b0c74)
![Screenshot-97](https://github.com/user-attachments/assets/260b29af-3f95-4ff7-8fe5-b021ace3b2f2)

