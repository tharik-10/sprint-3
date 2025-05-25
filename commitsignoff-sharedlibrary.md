![jenkinsgit2](https://github.com/user-attachments/assets/16efe3a1-64ed-4681-ac2f-e04e907deb24)

# **Jenkins Shared Library for Commit Sign-off (Declarative Pipeline Compatible)**

| Created    | Last updated | Version   | Author         | Internal Reviewer | L0     | L1          | L2              |
| ---------- | ------------ | --------- | -------------- | ----------------- | ------ | ----------- | --------------- |
| 2025-05-25 | 2025-05-25   | Version 1 | Mohamed Tharik | Priyanshu         | Khushi | Mukul Joshi | Piyush Upadhyay |

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Steps to Setup Jenkins Shared Library for Commit Sign-off](#steps-to-setup-jenkins-shared-library-for-commit-sign-off)
   - [Step 1: Create a Shared Library Repository](#step-1-create-a-shared-library-repository)  
   - [Step 2: Create the `commitSignoff.groovy` File](#step-2-create-the-commitsignoffgroovy-file)  
   - [Step 3: Use Shared Library in Jenkinsfile](#step-3-use-shared-library-in-jenkinsfile)  
   - [Step 4: Configure Shared Library in Jenkins UI](#step-4-configure-shared-library-in-jenkins-ui)  
   - [Step 5: Run the Pipeline](#step-5-run-the-pipeline)  
4. [Shared Library Content](#shared-library-content)
5. [Best Practices](#best-practices)
6. [Conclusion](#conclusion)
7. [Contact Information](#contact-information)
8. [References](#references)

## Introduction

Commit sign-off ensures contributions are traceable, legally acceptable, and responsible, aligning with DCO compliance. Moving to a Jenkins Shared Library architecture enables reuse, consistency, and optimization of Jenkins resources, allowing for DRY (Don't Repeat Yourself) pipeline practices across multiple repositories and teams.

## Prerequisites

| Requirement                      | Description                                          |
| -------------------------------- | ---------------------------------------------------- |
| Jenkins Server                   | Running instance with access rights                  |
| Git Repository                   | Source repositories requiring commit validation      |
| Shared Library Repository        | A GitHub repo to store the shared pipeline logic     |
| Git Installation                 | Git installed on Jenkins agents                      |
| Pipeline + Shared Library Plugin | Jenkins plugins installed (Pipeline, Shared Library) |

## Steps to Setup Jenkins Shared Library for Commit Sign-off

### Step 1: Create a Shared Library Repository

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

### Step 2: Create the commitSignoff.groovy File
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

### Step 3: Use Shared Library in Jenkinsfile
In your app repo (`tharik-10/sprint-3`), your `Jenkinsfile` should now look like:
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
### Step 4: Configure Shared Library in Jenkins UI
- Go to **Jenkins Dashboard > Manage Jenkins > Configure System**.
- Scroll to **Global Trusted Pipeline Libraries**.
- Add:
  - **Name**: `my-shared-lib`
  - **Default Version**: `main`
  - **Project Repository**: `https://github.com/tharik-10/jenkins-shared-library.git`
  - **Credentials** (if private): select the GitHub token
  
![Screenshot-97 (1)](https://github.com/user-attachments/assets/2ad582b8-d2a5-4fc8-87be-8fb3225a745f)

### Step 5: Run the Pipeline
Now when you run the pipeline, it should:
![Screenshot-100](https://github.com/user-attachments/assets/8c56aa72-02c4-401c-be29-1b6a2357bbf9)

- Checkout the code
![Screenshot-97](https://github.com/user-attachments/assets/0073d8ac-e080-415f-b04c-e17b26a5646c)

- Perform Git user config
- Make a dummy file change
- Sign-off commit
- Push to GitHub
- Print the commit message
![Screenshot-99](https://github.com/user-attachments/assets/78a06677-e14a-44bb-a8e9-ccbb777b389a)

## Shared Library Content

| File                        | Purpose                                                                      |
| --------------------------- | ---------------------------------------------------------------------------- |
| `vars/commitSignoff.groovy` | Handles all Git configuration, file change, commit, sign-off, and push logic |
| `vars/checkoutCode.groovy`  | Simple SCM checkout logic for reuse                                          |
| `resources/`                | Store templates or config files if needed                                    |

## Best Practices

| Best Practice                          | Explanation                                        |
| -------------------------------------- | -------------------------------------------------- |
| **Centralize common logic**            | Prevent script duplication across repositories     |
| **Parameterize all steps**             | Reusable functions with custom values              |
| **Fail early**                         | Validate commit sign-offs before CI/CD stages      |
| **Use credentials binding**            | Prevents secrets from leaking in logs              |
| **Version your library**               | Use tags/releases to lock stable versions          |
| **Use declarative syntax for clarity** | Declarative + shared library = readable + reusable |

## Conclusion

Using a Jenkins Shared Library for commit sign-off checks simplifies pipeline creation, enforces best practices across teams, and ensures DCO compliance efficiently. Shared libraries are ideal for reusing logic like SCM checkout, environment setup, artifact handling, and notifications.

## Contact Information

| Name           | Email address                                                                 |
| -------------- | ----------------------------------------------------------------------------- |
| Mohamed Tharik | [md.tharik.sanaatak@mygurukulam.co](mailto:md.tharik.sanaatak@mygurukulam.co) |

## References

| Link                                                                                                                                          | Description                                          |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)                                                        | How to use and configure Shared Libraries in Jenkins |
| [Developer Certificate of Origin](https://developercertificate.org/)                                                                          | DCO official explanation                             |
| [Git Commit Sign-off](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---signoff)                                             | Git commit `--signoff` usage                         |
| [GitHub DCO Checks](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-required-commit-signing) | How to enforce DCO in GitHub                         |
