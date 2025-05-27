![ci-bugs-java2](https://github.com/user-attachments/assets/6a53cf9c-ffd5-44af-9345-487d04ac42b7)

# **Jenkins Shared Libary Pipeline for Java Bugs Analysis**

## Define the Pipeline
```groovy
@Library('my-shared-lib@main') _

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

        stage('Common Stages') {
            steps {
                script {
                    commonStages()
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY}"
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
## Jenkins Shared Library Folder Directory and Script
```groovy
def call(Map config = [:]) {
    stage('Verify Code') {
        echo 'Listing files from copied repo...'
        sh 'ls -la'
    }

    stage('Build') {
        echo 'Building the salary-api (skipping tests)...'
        sh 'mvn clean install'
    }
}
```
![Screenshot-100](https://github.com/user-attachments/assets/de6af916-889b-40d4-adec-5c0e5a5e2f6b)

## Click Build Now to trigger the job.
![Screenshot-98 (3)](https://github.com/user-attachments/assets/1bfdd4f4-3deb-4f1d-8610-f8cc81d8aff7)

## Monitor the Console Output for build, analysis, and quality gate results.
![Screenshot-98 (2)](https://github.com/user-attachments/assets/635248f0-3255-404c-8ffb-0eeaeeba9179)
![Screenshot-99 (2)](https://github.com/user-attachments/assets/7c815c87-7ab6-4664-8a93-f141b02457a0)
![Screenshot-100 (2)](https://github.com/user-attachments/assets/f8fcd099-158d-49bf-a5d7-407b1a5b7153)
![Screenshot-101](https://github.com/user-attachments/assets/2ba23ed6-4479-44e9-9eea-8e20d8466329)

## Verify Results in SonarQube UI
![Screenshot-99 (1)](https://github.com/user-attachments/assets/2f2ae058-0a8e-4601-b90c-7524ff072e0a)
![Screenshot-98 (1)](https://github.com/user-attachments/assets/1d195b79-8a50-420c-a95e-a66a3eb1ccc1)
![Screenshot-98](https://github.com/user-attachments/assets/40352e78-d3d9-49c6-87fc-246218f02568)

