pipeline {
  agent any
  options { timestamps() }
  triggers { pollSCM('H/5 * * * *') } // poll every ~5 mins (no webhook needed)

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps {
        sh 'node -v && npm -v'
        sh 'npm ci || npm install'
      }
    }

    stage('Run Tests') {
      steps {
        script {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            sh 'npm test | tee test.log'
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'test.log', allowEmptyArchive: true
        }
        success {
          emailext(
            to: "obrisr@gmail.com",
            subject: "Tests SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Tests passed.\nSee attached test.log.",
            attachmentsPattern: 'test.log'
          )
        }
        failure {
          emailext(
            to: "obrisr@gmail.com",
            subject: "Tests FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Tests failed.\nSee attached test.log.",
            attachmentsPattern: 'test.log'
          )
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        sh 'npm run coverage | tee coverage.log || true'
      }
      post {
        always {
          archiveArtifacts artifacts: 'coverage.log, coverage/**', allowEmptyArchive: true
        }
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        script {
          catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
            sh 'npm audit --audit-level=low | tee npm-audit.txt'
            sh 'npm audit --json > npm-audit.json || true'
          }
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'npm-audit.*', allowEmptyArchive: true
        }
        success {
          emailext(
            to: "obrisr@gmail.com",
            subject: "Security Scan CLEAN: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "npm audit completed successfully.\nSee attachments.",
            attachmentsPattern: 'npm-audit.*'
          )
        }
        unstable {
          emailext(
            to: "obrisr@gmail.com",
            subject: "Security Scan Issues: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "Vulnerabilities detected by npm audit.\nSee attachments.",
            attachmentsPattern: 'npm-audit.*'
          )
        }
        failure {
          emailext(
            to: "obrisr@gmail.com",
            subject: "Security Scan FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: "npm audit did not run successfully.\nSee attachments.",
            attachmentsPattern: 'npm-audit.*'
          )
        }
      }
    }
  }
}
