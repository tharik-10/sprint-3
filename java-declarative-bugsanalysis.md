![ci-bugs-java2](https://github.com/user-attachments/assets/6a53cf9c-ffd5-44af-9345-487d04ac42b7)

# **Declarative Jenkins Pipeline for Java Bugs Analysis**

## Define the Pipeline
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
        stage('Clone Repository') {
            steps {
                echo '📥 Cloning salary-api repository...'
                // Clean workspace before cloning
                deleteDir()
                // Clone your GitHub repo
                git url: 'https://github.com/tharik-10/salary-api.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                echo '🔧 Building the salary-api (skipping tests)...'
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
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

## Click **Build Now** to trigger the job.
![Screenshot-98 (1)](https://github.com/user-attachments/assets/daaf346b-cd29-4a2f-91dd-af2a9b5d5783)

## Monitor the **Console Output** for build, analysis, and quality gate results.
![Screenshot-99](https://github.com/user-attachments/assets/943a07fe-3f60-4048-abba-8688573d54ec)
![Screenshot-98 (2)](https://github.com/user-attachments/assets/9911ab2b-02fa-44d3-ae57-1d35e791a5a3)
![Screenshot-98](https://github.com/user-attachments/assets/25a83010-d5bb-4346-ab46-90a81816b296)

## Verify Results in SonarQube UI 

![Screenshot-97](https://github.com/user-attachments/assets/67ef1d28-caee-40d4-a0e5-ab2ce5fa9071)

- Explore:
  - **Bugs**
  - **Code Smells**
  - **Security Hotspots**
  - **Duplications**
  - **Code Coverage (if configured)**
![Screenshot-97 (2)](https://github.com/user-attachments/assets/9bb42cde-8a1f-48c7-9309-6498eea4f8dd)
## We can See what are the issues we have in our Project or Code 
![Screenshot-98 (3)](https://github.com/user-attachments/assets/56aa2349-e9e4-4241-8efd-7bfaaf08583c)


