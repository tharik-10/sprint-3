![jenkinsgit2](https://github.com/user-attachments/assets/16efe3a1-64ed-4681-ac2f-e04e907deb24)

# **Jenkins Shared Library for Commit Sign-off (Declarative Pipeline Compatible)**

| Created    | Last updated | Version   | Author         | Internal Reviewer | L0     | L1          | L2              |
| ---------- | ------------ | --------- | -------------- | ----------------- | ------ | ----------- | --------------- |
| 2025-05-25 | 2025-05-25   | Version 2 | Mohamed Tharik | Priyanshu         | Khushi | Mukul Joshi | Piyush Upadhyay |

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Steps to Setup Jenkins Shared Library for Commit Sign-off](#steps-to-setup-jenkins-shared-library-for-commit-sign-off)
   * [Step 1: Create a Shared Library Repository](#step-1-create-a-shared-library-repository)
   * [Step 2: Add Common Scripts in `vars/`](#step-2-add-common-scripts-in-vars)
   * [Step 3: Configure Jenkins to Use the Shared Library](#step-3-configure-jenkins-to-use-the-shared-library)
   * [Step 4: Use the Shared Library in Jenkinsfile](#step-4-use-the-shared-library-in-jenkinsfile)
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
│   ├── checkoutCode.groovy
├── resources/
│   └── (optional static templates)
├── src/
│   └── org/myorg/utils/... (if needed)
└── README.md
```

### Step 2: Add Common Scripts in `vars/`

**Example: vars/commitSignoff.groovy**

```groovy
def call(String username, String email, String message, String credentialsId, String repoUrl, String branch = 'main') {
    sh """
        git config user.name '${username}'
        git config user.email '${email}'
        echo "Pipeline ran on: \$(date)" > pipeline-log.txt
        git add pipeline-log.txt
        git commit -m '${message}' -s || echo 'No changes to commit.'
    """

    withCredentials([usernamePassword(credentialsId: credentialsId, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        sh '''
            git remote set-url origin https://${USERNAME}:${PASSWORD}@${repoUrl}
            git push origin HEAD:${branch} || echo "Nothing to push."
        '''
    }
    echo "✅ Commit sign-off and push completed."
}
```

**Example: vars/checkoutCode.groovy**

```groovy
def call() {
    checkout scm
    echo "✅ Code checkout completed."
}
```

### Step 3: Configure Jenkins to Use the Shared Library

* Navigate to **Manage Jenkins → Configure System**.
* Under **Global Pipeline Libraries**, click **Add**:

  * **Name**: `my-shared-lib`
  * **Default Version**: `main` or a tag
  * **Project Repository**: `https://github.com/tharik-10/jenkins-shared-library.git`
  * **Credentials**: If private, use your GitHub token credentials

### Step 4: Use the Shared Library in Jenkinsfile

**Declarative Jenkinsfile Example:**

```groovy
@Library('my-shared-lib') _

def GIT_USER = 'tharik-10'
def GIT_EMAIL = 'md.tharik@mygurukulam.co'
def COMMIT_MESSAGE = 'Demo commit using shared library'
def CRED_ID = 'github-token1'
def REPO_URL = 'github.com/tharik-10/sprint-3.git'

declarative {
  pipeline {
    agent any

    stages {
      stage('Checkout') {
        steps {
          checkoutCode()
        }
      }

      stage('Commit Sign-off') {
        steps {
          commitSignoff(GIT_USER, GIT_EMAIL, COMMIT_MESSAGE, CRED_ID, REPO_URL)
        }
      }
    }

    post {
      success {
        echo '✅ Build succeeded with commit sign-off.'
      }
      failure {
        echo '❌ Build failed.'
      }
    }
  }
}
```
### Step 5: Test the Shared Library
- Create a test GitHub repository with a `Jenkinsfile` that uses the shared library functions `(checkoutCode()` and `commitSignoff()`).
- Set up a Jenkins pipeline job pointing to the test repo.
- Trigger the job and verify:
  - The code is checked out successfully.
  - A signed-off commit is created and pushed.
- Check the GitHub repo to confirm the new commit with `Signed-off-by` in the message.
- Ensure credentials are correct and changes exist to trigger a commit.

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
