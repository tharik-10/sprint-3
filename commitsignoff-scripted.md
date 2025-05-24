![jenkinsgit2](https://github.com/user-attachments/assets/16efe3a1-64ed-4681-ac2f-e04e907deb24)

# **Scripted Jenkins Pipeline for Commit Sign-off**
| Created        | Last updated      | Version         | author|  Internal Reviewer | L0 | L1 | L2|
|----------------|----------------|-----------------|-----------------|-----|------|----|----|
| 2025-05-24  | 2025-05-24   |     Version 1         |  Mohamed Tharik |Priyanshu|Khushi|Mukul Joshi |Piyush Upadhyay|

## Table of Contents 

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Steps to Setup Scripted Pipeline for Commit Sign-off](#steps-to-setup-scripted-pipeline-for-commit-sign-off)
   - [Step 1: Ensure Your Repository Has a Jenkinsfile](#step-1-ensure-your-repository-has-a-jenkinsfile)
   - [Step 2: Add GitHub Credentials in Jenkins](#step-2-add-github-credentials-in-jenkins)
   - [Step 3: Create a New Jenkins Pipeline Job](#step-3-create-a-new-jenkins-pipeline-job)
   - [Step 4: What the Pipeline Does](#step-4-what-the-pipeline-does)
   - [Step 5: Create Jenkinsfile in Root Directory of Repository](#step-5-create-jenkinsfile-in-root-directory-of-repository)
   - [Step 6: Run the Pipeline](#step-6-run-the-pipeline)
4. [Best Practices](#best-practices)
5. [Conclusion](#conclusion)
6. [Contact Information](#contact-information)
7. [References](#references)

## Introduction 
Commit sign-off is a lightweight certification that ensures contributors are submitting code they have the right to share, often used in open-source and enterprise environments to comply with the Developer Certificate of Origin (DCO). This document outlines how to create a Scripted Jenkins Pipeline that validates commit sign-offs and runs generic CI checks, helping teams automate compliance and maintain code quality across all contributions.

## Prerequisites
| Requirement              | Description                                                       |
| ------------------------ | ----------------------------------------------------------------- |
| Jenkins Server           | A running Jenkins instance with proper access rights              |
| Git Repository           | Source code repository (e.g., GitHub, GitLab) with commit history |
| Pipeline Plugin          | Jenkins Pipeline plugin installed                                 |
| Git Installation         | Git should be installed and available on Jenkins agents           |
| SCM Integration          | Jenkins configured with credentials to access the Git repository  |
| Basic Git Knowledge      | Understanding of git commits and `Signed-off-by` convention       |

## Steps to Setup Scripted Pipeline for Commit Sign-off
### Step 1: Ensure Your Repository Has a Jenkinsfile
- Place the pipeline script (`Jenkinsfile`) at the root of your GitHub repository.
### Step 2: Add GitHub Credentials in Jenkins
- In Jenkins, navigate to **Manage Jenkins → Credentials**.
- Add **Username and Password** type credentials:
  - **Username**: Your GitHub username (e.g., `tharik-10`)
  - **Password**: Your GitHub Personal Access Token (PAT)
- Assign an ID to these credentials, e.g., `github-token1`.

### Step 3: Create a New Jenkins Pipeline Job
- Create a new job in Jenkins.
- Choose **Pipeline** as the job type.
- Configure the pipeline to load from your GitHub repository using SCM.
- Provide the repository URL, branch (e.g., `main`), and credentials (`github-token1`).
- Ensure `Jenkinsfile` is set as the script path.
### Step 4: What the Pipeline Does
| Stage                | Description                                            |
| -------------------- | ------------------------------------------------------ |
| Checkout Code        | Clones the repository                                  |
| Configure Git        | Sets up Git username/email in the workspace            |
| Make Dummy Change    | Writes a timestamp to `pipeline-log.txt` and stages it |
| Commit with Sign-off | Commits with `-s` flag to add `Signed-off-by` line     |
| Print Commit Message | Displays the latest commit message in the console      |
| Push Changes         | Pushes to GitHub using secured credentials             |
### Step 5: Create Jenkinsfile in Root Directory of Repository
```bash
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
```
### Step 6: Run the Pipeline
- Click Build Now on your Jenkins job.
- Monitor the console output to see:
  - The commit message with sign-off.
  - Confirmation that the push succeeded or a message indicating there's nothing to push.

## Best Practices
| Best Practice                          | Explanation                                                                |
| -------------------------------------- | -------------------------------------------------------------------------- |
| **Always check against a base branch** | Use `origin/main` or `origin/master` as the base for comparing new commits |
| **Fail early**                         | Catch missing sign-offs before running full CI to save resources           |
| **Display clear messages**             | Make it easy for contributors to understand what failed and how to fix it  |
| **Enforce sign-off via hooks**         | Optionally enforce with pre-commit hooks or GitHub branch protection rules |
| **Document contribution guidelines**   | Include DCO and sign-off process in `CONTRIBUTING.md`                      |
| **Use Scripted Pipeline for Fine Control**   | Scripted Pipeline allows granular control over each step, perfect for enforcing custom logic like DCO. |

## Conclusion
Implementing a Scripted Jenkins Pipeline with Commit Sign-off validation is a robust step toward securing your CI/CD processes. It ensures contributions are legitimate and traceable while keeping pipelines clean, standardized, and automated. Combining this with automated CI checks strengthens your development lifecycle by promoting responsibility, legal clarity, and quality assurance.

## Contact Information
| Name           | Email address                                                                 |
| -------------- | ----------------------------------------------------------------------------- |
| Mohamed Tharik | [md.tharik.sanaatak@mygurukulam.co](mailto:md.tharik.sanaatak@mygurukulam.co) |

## References
| Link                                                                                                                                          | Description                                                         |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| [Jenkins Pipelines](https://www.jenkins.io/doc/book/pipeline/)                                                                                | Official Jenkins documentation for creating and managing pipelines. |
| [Developer Certificate of Origin](https://developercertificate.org/)                                                                          | Explains the DCO and its purpose in open-source contributions.      |
| [Git Commit Sign-off](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---signoff)                                             | Git documentation explaining the `--signoff` option during commits. |
| [GitHub DCO Checks](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-required-commit-signing) | Guide for enabling and enforcing DCO checks on GitHub repositories. |
| [Scripted Pipeline Syntax (Official Jenkins Docs)](https://www.jenkins.io/doc/book/pipeline/syntax/#scripted-pipeline) | How Scripted Pipeline works and Syntax of it.|
