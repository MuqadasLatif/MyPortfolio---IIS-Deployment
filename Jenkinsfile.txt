pipeline {
    agent any

    environment {
        DEPLOY_DIR = 'C:\\inetpub\\MyPortfolio'
        SITE_URL   = 'http://localhost:8082'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to IIS') {
            steps {
                powershell '''
                    Copy-Item -Path "$env:WORKSPACE\\index.html", "$env:WORKSPACE\\style.css" -Destination $env:DEPLOY_DIR -Force
                '''
            }
        }

        stage('Verify') {
            steps {
                powershell '''
                    $response = Invoke-WebRequest -Uri $env:SITE_URL -UseBasicParsing
                    if ($response.StatusCode -ne 200) { exit 1 }
                    Write-Output "Site is live: $env:SITE_URL"
                '''
            }
        }
    }

    post {
        success { echo 'Deployment successful!' }
        failure { echo 'Deployment failed. Check the console output.' }
    }
}
