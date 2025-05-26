# **Declarative Jenkins Pipeline for Java Bugs Analysis**
| Created        | Last updated      | Version         | author|  Internal Reviewer | L0 | L1 | L2|
|----------------|----------------|-----------------|-----------------|-----|------|----|----|
| 2025-05-26  | 2025-05-26   |     Version 1         |  Mohamed Tharik |Priyanshu|Khushi|Mukul Joshi |Piyush Upadhyay|

## Table of Contents 

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
    maven 'Maven 3.9.2'   // Make sure this name matches your Maven tool name in Jenkins
    jdk 'JDK11'           // Make sure this matches your installed JDK in Jenkins
  }

  environment {
    SONAR_PROJECT_KEY = 'salary-api'
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/OT-MICROSERVICES/salary-api.git'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean install'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY}'
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }
}
```
### Step 3: Save and Run
- Click **Save**.
- Click **Build Now** to trigger the job.
- Monitor the **Console Output** for build, analysis, and quality gate results.

### Step 4: Verify Results in SonarQube
- Log in to your SonarQube UI.
- Locate the project salary-api.
- Explore:
  - **Bugs**
  - **Code Smells**
  - **Security Hotspots**
  - **Duplications**
  - **Code Coverage (if configured)**

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

| Link                                                                                                                                          | Description                                                                 |
|-----------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
