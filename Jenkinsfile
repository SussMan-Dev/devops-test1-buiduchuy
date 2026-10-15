def sendTelegram(String message) {
    withCredentials([
        string(credentialsId: 'telegram-token', variable: 'BOT_TOKEN'),
        string(credentialsId: 'telegram-chat-id', variable: 'CHAT_ID')
    ]) {
        withEnv(["TELEGRAM_MESSAGE=${message}"]) {
            powershell '''
                $body = @{
                    chat_id    = $env:CHAT_ID
                    text       = $env:TELEGRAM_MESSAGE
                    parse_mode = 'Markdown'
                } | ConvertTo-Json

                Invoke-RestMethod `
                    -Uri "https://api.telegram.org/bot$env:BOT_TOKEN/sendMessage" `
                    -Method Post `
                    -ContentType 'application/json' `
                    -Body $body
            '''
        }
    }
}

pipeline {
    agent any

    tools {
        nodejs 'NodeJS 24.21.0'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source...'
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                echo 'Installing dependencies...'
                bat 'npm ci'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project...'
                bat 'npm run build'
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sendTelegram("""**Deploy Started**

🚀 DEPLOY STARTED
Project: devops-test
Branch: main""")
                }

                echo 'Deploying to Netlify...'

                withCredentials([
                    string(credentialsId: 'netlify-token', variable: 'NETLIFY_AUTH_TOKEN'),
                    string(credentialsId: 'netlify-site-id', variable: 'NETLIFY_SITE_ID')
                ]) {
                    bat 'npx netlify deploy --prod --dir=dist --site %NETLIFY_SITE_ID% --auth %NETLIFY_AUTH_TOKEN%'
                }
            }
        }
    }

    post {
        success {
            script {
                sendTelegram("""**Deploy Success**

✅ DEPLOY SUCCESS NHÉ
Project: devops-test
Branch: main
URL: https://gorgeous-puppy-f0b50d.netlify.app/""")
            }
        }

        failure {
            script {
                sendTelegram("""**Deploy Failed**

❌ DEPLOY FAILED RỒI
Project: devops-test
Branch: main

Please check Jenkins.""")
            }
        }
    }
}
