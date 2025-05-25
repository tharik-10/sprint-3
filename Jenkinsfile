@Library('my-shared-lib@main') _

def GIT_USER = 'tharik-10'
def GIT_EMAIL = 'md.tharik@mygurukulam.co'
def COMMIT_MESSAGE = 'Demo commit using shared library'
def CRED_ID = 'github-token1'
def BRANCH = 'main'  // Optional if you want to keep it dynamic

pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        script {
          checkoutCode()
        }
      }
    }

    stage('Print Commit Message') {
      steps {
        script {
          printCommitMessage()
        }
      }
    }

    stage('Commit Sign-off') {
      steps {
        script {
          commitSignoff(GIT_USER, GIT_EMAIL, COMMIT_MESSAGE, CRED_ID, BRANCH)
        }
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

