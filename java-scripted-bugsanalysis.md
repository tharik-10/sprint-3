![ci-bugs-java2](https://github.com/user-attachments/assets/6a53cf9c-ffd5-44af-9345-487d04ac42b7)

# **Scripted Jenkins Pipeline for Java Bugs Analysis**

## Define the Pipeline
```groovy
node {
    // Define tools
    def mvnHome = tool name: 'Maven 3.9.2', type: 'maven'
    def jdkHome = tool name: 'JDK17', type: 'jdk'
    env.PATH = "${jdkHome}/bin:${mvnHome}/bin:${env.PATH}"
    env.SONAR_PROJECT_KEY = 'salary-api'

    try {
        stage('Verify Code') {
            echo 'Listing files from copied repo...'
            sh 'ls -la'
        }

        stage('Build') {
            echo 'Building the salary-api (skipping tests)...'
            sh "${mvnHome}/bin/mvn clean install"
        }

        stage('SonarQube Analysis') {
            echo 'Running SonarQube analysis...'
            withSonarQubeEnv('SonarQube') {
                sh "${mvnHome}/bin/mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY}"
            }
        }

        stage('SonarQube Quality Gate') {
            timeout(time: 5, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }

        echo '🎉 Build succeeded!'
    } catch (err) {
        echo "❌ Build failed! Reason: ${err}"
        currentBuild.result = 'FAILURE'
    }
}
```
## Click **Build Now** to trigger the job.
![Screenshot-98](https://github.com/user-attachments/assets/e378537d-3473-423a-a428-7c2983736301)

## Monitor the **Console Output** for build, analysis, and quality gate results.
![Screenshot-98 (1)](https://github.com/user-attachments/assets/6068c9a5-f1ed-48a7-996c-9335058a40e1)
![Screenshot-99](https://github.com/user-attachments/assets/ecaae723-fadd-4edc-80d7-64192cf93580)
![Screenshot-100](https://github.com/user-attachments/assets/c9335adf-54e6-4c20-a6d0-8985bed029e1)
![Screenshot-101](https://github.com/user-attachments/assets/0ea50ce1-dfbd-4438-8cc6-ad2205b32da2)

## Verify Results in SonarQube UI 

- Explore:
  - **Bugs**
  - **Code Smells**
  - **Security Hotspots**
  - **Duplications**
  - **Code Coverage (if configured)**
![Screenshot-98 (2)](https://github.com/user-attachments/assets/3a3c58b5-2e01-482a-a167-a13abe13dcaa)
