![jenkinsgit2](https://github.com/user-attachments/assets/16efe3a1-64ed-4681-ac2f-e04e907deb24)

# **Declarative Jenkins Pipeline for Commit Sign-off**
| Created        | Last updated      | Version         | author|  Internal Reviewer | L0 | L1 | L2|
|----------------|----------------|-----------------|-----------------|-----|------|----|----|
| 2025-05-24  | 2025-05-24   |     Version 1         |  Mohamed Tharik |Priyanshu|Khushi|Mukul Joshi |Piyush Upadhyay|

## Table of Contents 

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Steps to Setup Declarative Pipeline for Commit Sign-off](#steps-to-setup-declarative-pipeline-for-commit-sign-off)
   - [Step 1: Make sure your repo has a Jenkinsfile](#step-1-make-sure-your-repo-has-a-jenkinsfile)
   - [Step 2: Add GitHub credentials in Jenkins](#step-2-add-github-credentials-in-jenkins)
   - [Step 3: Create a new Jenkins Pipeline job](#step-3-create-a-new-jenkins-pipeline-job)
   - [Step 4: What the Pipeline Does](#step-4-what-the-pipeline-does)
   - [Step 5: Create Jenkinsfile in Root Directory of Repository Using Declarative Pipeline](#step-5-create-jenkinsfile-in-root-directory-of-repository-using-declarative-pipeline)
   - [Step 6: Run the Pipeline](#step-6-run-the-pipeline)
4. [Best Practices](#best-practices)
5. [Conclusion](#conclusion)
6. [Contact Information](#contact-information)
7. [References](#references)

## Introduction 
Commit sign-off is a lightweight certification that ensures contributors are submitting code they have the right to share, often used in open-source and enterprise environments to comply with the Developer Certificate of Origin (DCO). This document outlines how to create a Declarative Jenkins Pipeline that validates commit sign-offs and runs generic CI checks, helping teams automate compliance and maintain code quality across all contributions.

## Prerequisites

| Requirement                  | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Jenkins Server              | A running Jenkins instance with proper access rights                        |
| Git Repository              | Source code repository (e.g., GitHub, GitLab) with commit history           |
| Pipeline Plugin             | Jenkins Pipeline plugin installed                                           |
| Git Installation            | Git should be installed and available on Jenkins agents                    |
| SCM Integration             | Jenkins configured with credentials to access the Git repository            |
| Basic Git Knowledge         | Understanding of git commits and `Signed-off-by` convention                 |
| Jenkins Agent (Optional)    | A configured Jenkins agent for scalable build execution                     |

## Steps to Setup Declarative Pipeline for Commit Sign-off
### Step 1: Make sure your repo has a Jenkinsfile
- Put the pipeline script (Jenkinsfile) at the root of your GitHub repo.

### Step 2: Add GitHub credentials in Jenkins
- In Jenkins, go to **Manage Jenkins → Credentials**.
- Add **Username and Password** type credentials:
  - Username = your GitHub username (e.g., `tharik-10`)
  - Password = your GitHub Personal Access Token (PAT)
- Give it an ID, e.g., `github-token1`.

### Step 3: Create a new Jenkins Pipeline job
- Create a new job in Jenkins.
- Choose **Pipeline** as the job type.
- Set the pipeline to load from your GitHub repo using SCM.
- Use the repo URL, branch (main), and credentials (`github-token1`).
- Make sure `Jenkinsfile` is set as the script path.

![commitsignoff-declarative2](https://github.com/user-attachments/assets/2056287e-da36-4ece-96bf-d04b520cd0a0)

### Step 4: What the Pipeline Does

| Stage                | Description                                            |
| -------------------- | ------------------------------------------------------ |
| Checkout Code        | Clones the repo                                        |
| Configure Git        | Sets up Git username/email in the workspace            |
| Make Dummy Change    | Writes a timestamp to `pipeline-log.txt` and stages it |
| Commit with Sign-off | Commits with `-s` flag to add `Signed-off-by` line     |
| Print Commit Message | Displays the full latest commit message in the console |
| Push Changes         | Pushes to GitHub using secured credentials             |
![commitsign-declarative3](https://github.com/user-attachments/assets/0f0e6b2d-21ae-4ab1-9ff2-8e96c43db665)
![commitsignoff-declarative4](https://github.com/user-attachments/assets/f82997b9-d394-4c83-b792-ed66521537f1)

### Step 5: Create Jenkinsfile in Root Directory of Repository Using Declarative Pipeline
```bash
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
                    sh '''
                    git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/tharik-10/sprint-3.git
                    git push origin HEAD:main || echo "Nothing to push."
                    '''
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
```
### Step 6: Run the Pipeline
- Click **Build Now** on your Jenkins job.

![commitsignoff-declarative](https://github.com/user-attachments/assets/441bfd16-48ed-4902-b3d4-6930cca96a68)

- Watch the console output to see:
  - Commit message with sign-off.
  
![commitsignoff-declarative5](https://github.com/user-attachments/assets/2e144788-80e1-4db0-b9b2-b5e994663c50)

  - Confirmation that push succeeded or if nothing to push.

![commitsignoff-declarative6](https://github.com/user-attachments/assets/32a23551-b62a-431e-bb9f-f7e4010b0c74)
![Screenshot-97](https://github.com/user-attachments/assets/260b29af-3f95-4ff7-8fe5-b021ace3b2f2)

## Best Practices
| Best Practice                            | Explanation                                                                                                                          |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Always check against a base branch** | Use `origin/main` or `origin/master` as the base for comparing new commits                                                           |
| **Fail early**                         | Catch missing sign-offs before running full CI to save resources                                                                     |
| **Display clear messages**             | Make it easy for contributors to understand what failed and how to fix it                                                            |
| **Enforce sign-off via hooks**         | Optionally enforce with pre-commit hooks or GitHub branch protection rules                                                           |
| **Document contribution guidelines**   | Include DCO and sign-off process in `CONTRIBUTING.md`                                                                                |
| **Use Declarative Pipeline for Readability** | Declarative syntax provides a structured and easy-to-read pipeline for team collaboration |

## Conclusion
Implementing a Declarative Jenkins Pipeline with Commit Sign-off validation is a robust step toward securing your CI/CD processes. It ensures contributions are legitimate and traceable while keeping pipelines clean, standardized, and automated. Combining this with automated CI checks strengthens your development lifecycle by promoting responsibility, legal clarity, and quality assurance.

## Contact Information
| Name | Email address         |
|------|------------------------|
| Mohamed Tharik  | md.tharik.sanaatak@mygurukulam.co    |

## References

| Link                                                                                                                                          | Description                                                                 |
|-----------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| [Jenkins Pipelines](https://www.jenkins.io/doc/book/pipeline/)                                                                                | Official Jenkins documentation for creating and managing pipelines.        |
| [Developer Certificate of Origin](https://developercertificate.org/)                                                                          | Explains the DCO and its purpose in open-source contributions.             |
| [Git Commit Sign-off](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---signoff)                                             | Git documentation explaining the `--signoff` option during commits.        |
| [GitHub DCO Checks](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-required-commit-signing) | Guide for enabling and enforcing DCO checks on GitHub repositories.        |
