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
        echo "📦 Checking out backend source..."
        checkout scm
      }
    }

    stage('Environment Setup') {
      steps {
        echo "🐍 Setting up Python virtual environment..."
        sh 'python3 -m venv .venv'
      }
    }

    stage('Install Requirements') {
      steps {
        echo "📥 Installing dependencies..."
        sh '.venv/bin/pip install --upgrade pip'
        sh '.venv/bin/pip install -r requirements.txt'
      }
    }

    stage('Database Migrations') {
      steps {
        echo "🗄️ Running Django migrations..."
        sh '.venv/bin/python manage.py makemigrations'
        sh '.venv/bin/python manage.py migrate'
      }
    }

    stage('Unit Tests') {
      steps {
        echo "🧪 Running Django unit tests..."
        sh '.venv/bin/python manage.py test'
      }
    }

    stage('Start Server') {
      steps {
        echo "🚀 Starting development server (temporary)..."
        sh '.venv/bin/python manage.py runserver 0.0.0.0:8000 &'
        sh 'sleep 5'
      }
    }

    stage('API Test') {
      steps {
        echo "🔍 Running API test script..."
        sh '.venv/bin/python check.py'
      }
    }
  }

  post {
    success {
      echo "✅ Tests passed, triggering deployment..."
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

      mail to: 'xaioene@gmail.com',
           subject: "✅ Jenkins Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) completed successfully.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Deployment API triggered successfully.

Regards,  
Jenkins 🤖
"""
    }

    failure {
      echo "❌ Pipeline failed, sending error email..."
      mail to: 'xaioene@gmail.com',
           subject: "❌ Jenkins Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hey there,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) has failed.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Please check the console output for details.

– Jenkins 🧠
"""
    }
  }
}
