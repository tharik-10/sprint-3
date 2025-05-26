![ci-bugs-java2](https://github.com/user-attachments/assets/6a53cf9c-ffd5-44af-9345-487d04ac42b7)

# **Declarative Jenkins Pipeline for Java Bugs Analysis**
| Created        | Last updated      | Version         | author|  Internal Reviewer | L0 | L1 | L2|
|----------------|----------------|-----------------|-----------------|-----|------|----|----|
| 2025-05-26  | 2025-05-26   |     Version 1         |  Mohamed Tharik |Priyanshu|Khushi|Mukul Joshi |Piyush Upadhyay|

## Table of Contents 

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Steps to Set Up Java CI Pipeline for Bug Analysis](#steps-to-set-up-java-ci-pipeline-for-bug-analysis)
   - [Step 1: Create a New Pipeline Job](#step-1-create-a-new-pipeline-job)
   - [Step 2: Define the Pipeline](#step-2-define-the-pipeline)
   - [Step 3: Save and Run](#step-3-save-and-run)
   - [Step 4: Verify Results in SonarQube](#step-4-verify-results-in-sonarqube)
4. [Best Practices](#best-practices)
5. [Conclusion](#conclusion)
6. [Contact Information](#contact-information)
7. [References](#references)

## Introduction 
Continuous Integration (CI) ensures that code changes are automatically built, tested, and verified early in the development process. This document outlines the implementation of a CI pipeline for Java-based salary-api using a Declarative Jenkins Pipeline. It emphasizes static bug analysis using SonarQube, which provides in-depth insights into code quality, bugs, vulnerabilities, and code smells.

## Prerequisites
| Component                | Requirement / Description                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| **Jenkins**              | Installed and running (version 2.319+ recommended)                                         |
| **Java**                 | JDK 11 or higher installed on Jenkins agents                                               |
| **Maven**                | Maven 3.x installed and configured in Jenkins (e.g., `Maven 3`)                            |
| **Git**                  | Installed on Jenkins and agents                                                            |
| **Jenkins Plugins**      | Pipeline, Git, Maven Integration, SonarQube Scanner, Warnings Next Generation (optional)   |
| **SonarQube Server**     | Installed and running (local or hosted); reachable by Jenkins                              |
| **SonarQube Token**      | Token generated for authentication; added in Jenkins credentials as "Secret Text"          |
| **SonarQube in Jenkins** | SonarQube server configured in Jenkins (`Manage Jenkins → Configure System → SonarQube`)   |
| **Jenkins Credentials**  | Secret text credential added with SonarQube token (e.g., ID: `sonarqube-token`)            |
| **SonarQube Scanner**    | SonarQube Scanner tool installed in Jenkins global tools (e.g., name: `SonarQube Scanner`) |

## Steps to Set Up Java CI Pipeline for Bug Analysis
### Step 1: Create a New Pipeline Job
- Navigate to **Jenkins Dashboard**.
- Click **New Item**.
- Enter a name, e.g., `salary-api-sonarqube-pipeline`.
- Select **Pipeline**.
- Click **OK**.

### Step 2: Define the Pipeline
- Scroll to the **Pipeline** section.
- Choose **Pipeline script** and paste the following code:
```groovy
pipeline {
    agent any

    tools {
        maven 'Maven 3.9.2'
        jdk 'JDK17'
    }

    environment {
        SONAR_PROJECT_KEY = 'salary-api'
    }

    stages {
        stage('Verify Code') {
            steps {
                echo 'Listing files from copied repo...'
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the salary-api (skipping tests)...'
                sh 'mvn clean install -DskipTests=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY}'
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Build succeeded!'
        }

        failure {
            echo '❌ Build failed!'
        }
    }
}

```
### Step 3: Save and Run
- Click **Save**.
- Click **Build Now** to trigger the job.
![Screenshot-97 (1)](https://github.com/user-attachments/assets/9fe8f025-50c8-43ed-8dd4-e4da804be210)

- Monitor the **Console Output** for build, analysis, and quality gate results.
![Screenshot-99](https://github.com/user-attachments/assets/943a07fe-3f60-4048-abba-8688573d54ec)
![Screenshot-98](https://github.com/user-attachments/assets/25a83010-d5bb-4346-ab46-90a81816b296)

### Step 4: Verify Results in SonarQube
- Log in to your SonarQube UI.
- Locate the project salary-api.
![Screenshot-97](https://github.com/user-attachments/assets/67ef1d28-caee-40d4-a0e5-ab2ce5fa9071)

- Explore:
  - **Bugs**
  - **Code Smells**
  - **Security Hotspots**
  - **Duplications**
  - **Code Coverage (if configured)**
![Screenshot-97 (2)](https://github.com/user-attachments/assets/9bb42cde-8a1f-48c7-9309-6498eea4f8dd)

## Best Practices
| Best Practice                             | Description                                                                                    |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Use descriptive project keys            | Use meaningful project keys in SonarQube (e.g., `salary-api`)                                  |
| Ensure binaries are available           | Make sure compiled Java class files exist in `target/classes` before SonarQube analysis        |
| Secure tokens using Jenkins credentials | Store SonarQube tokens securely using Jenkins "Secret Text" credentials                        |
| Fail pipeline if Quality Gate fails     | Configure the pipeline to abort if SonarQube's Quality Gate fails (using `waitForQualityGate`) |
| Schedule periodic scans                 | Run scheduled static analysis to detect issues even when no recent code commits have occurred  |

## Conclusion
By integrating **SonarQube** with a **Declarative Jenkins Pipeline**, this CI implementation ensures that the `salary-api` Java microservice undergoes:
- Automatic build and test
- Real-time static bug analysis and reporting
- Artifact archival for delivery pipelines
This setup improves code reliability, helps catch regressions early, and promotes high development standards across the OT-MICROSERVICES project.

## Contact Information
| Name | Email address         |
|------|------------------------|
| Mohamed Tharik  | md.tharik.sanaatak@mygurukulam.co    |

## References

| Link                                                                                                        | Description                                        |
|-------------------------------------------------------------------------------------------------------------|----------------------------------------------------|
| [Jenkins Documentation](https://www.jenkins.io/doc/)                                                       | Official Jenkins documentation                     |
| [SonarQube Documentation](https://docs.sonarsource.com/)                                                   | Official SonarQube documentation                   |
| [Maven Documentation](https://maven.apache.org/guides/index.html)                                          | Apache Maven build tool guides                     |
| [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)                                | Declarative pipeline syntax and examples           |
| [SonarScanner for Maven](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner-for-maven/) | Using SonarScanner with Maven                      |
| [Git Documentation](https://git-scm.com/doc)                                                               | Git version control system                         |
