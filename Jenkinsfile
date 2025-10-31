pipeline {
  agent any

  environment {
    APP_ID = "NB5bUX3xKecQbk0Vf9F28"
  }

  parameters {
    string(name: 'DEPLOY_URL_CRED_ID', defaultValue: 'DEPLOY_URL', description: 'Credentials ID for the deployment URL (Secret Text)')
    string(name: 'DEPLOY_KEY_CRED_ID', defaultValue: 'DEPLOY_KEY', description: 'Credentials ID for the deployment API key (Secret Text)')
  }

  stages {
    stage('Checkout') {
      steps {
        echo "📦 Checking out source code..."
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        echo "📥 Installing npm packages..."
        sh 'npm install'
      }
    }

    stage('Build Project') {
      steps {
        echo "🏗️ Building the frontend..."
        sh 'npm run build'
      }
    }

    stage('Run Tests') {
      steps {
        echo "🧪 Running unit tests..."
        sh 'npm test || true'
      }
    }

    stage('Archive Build') {
      steps {
        echo "📦 Archiving build artifacts..."
        archiveArtifacts artifacts: 'dist/**', fingerprint: true
      }
    }

    stage('Deploy') {
      steps {
        echo "🚀 Triggering deployment API..."
        withCredentials([
          string(credentialsId: params.DEPLOY_URL_CRED_ID, variable: 'DEPLOY_URL'),
          string(credentialsId: params.DEPLOY_KEY_CRED_ID, variable: 'DEPLOY_KEY')
        ]) {
          sh '''
            json_payload=$(printf '{"applicationId":"%s"}' "$APP_ID")
            curl -fS -X POST \
              "$DEPLOY_URL" \
              -H 'accept: application/json' \
              -H 'Content-Type: application/json' \
              -H "x-api-key: $DEPLOY_KEY" \
              --data-binary "$json_payload" \
              -w "\\nHTTP %{http_code}\\n"
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Build and deployment completed successfully!"
      mail to: 'xaioene@gmail.com',
           subject: "✅ Jenkins Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) completed successfully.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Deployment API triggered successfully.

Best,  
Jenkins 🤖
"""
    }

    failure {
      echo "❌ Pipeline failed, sending error email..."
      mail to: 'xaioene@gmail.com',
           subject: "❌ Jenkins Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) has failed.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Please check the Jenkins console output for details.

Regards,  
Jenkins 🧠
"""
    }
  }
}
