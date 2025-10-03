### Jenkins Mengirim Notifikasi ke Telegram setelah Build Selesai
Pada case ini, Jenkins akan mengirimkan notifikasi ke Telegram setiap kali ada build yang selesai, baik itu sukses maupun gagal. Notifikasi ini akan berisi informasi seperti nama project, branch, commit, SHA, nomor build, status build, dan link ke log build.

Pada pipeline berikut, perlu disiapkan beberapa credential pada Jenkins:
- Credentials (dengan tipe "Secret text"):
  - `telegram-bot-id`: ID/Token bot Telegram
  - `telegram-chat-id`: ID chat grup Telegram
  - `telegram-topic-id`: ID topic pada grup Telegram

![enter image description here](https://i.imgur.com/WPD7O0i_d.webp?maxwidth=760&fidelity=grand)
Pada pipeline jenkinsfile berikut, setelah semua stage selesai, akan ada blok `post` yang akan selalu dijalankan (`always`). Di dalam blok ini, kita mengambil pesan commit terakhir dan mengirimkan notifikasi ke Telegram menggunakan `curl` ke API bot Telegram.
```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = "frontend-test"
        DOCKERHUB_REPO = "adkurnwn/frontend-test"
        BUILD_TAG      = "dev-actions-${env.GIT_COMMIT}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev/actions', url: 'https://github.com/kurww/frontend-test.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_TAG .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        docker tag $DOCKER_IMAGE:$BUILD_TAG $DOCKERHUB_REPO:$BUILD_TAG

                        docker push $DOCKERHUB_REPO:$BUILD_TAG
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                def commitMessage = sh(script: 'git log -1 --pretty=%s', returnStdout: true).trim()
                withCredentials([
                    string(credentialsId: 'telegram-bot-id', variable: 'TELEGRAM_BOT_ID'),
                    string(credentialsId: 'telegram-chat-id', variable: 'TELEGRAM_CHAT_ID'),
                    string(credentialsId: 'telegram-topic-id', variable: 'TELEGRAM_TOPIC_ID')
                ]) {
                    def statusMessage
                    def emoji

                    if (currentBuild.currentResult == 'SUCCESS') {
                        statusMessage = "BERHASIL"
                        emoji = "✅"
                    } else {
                        statusMessage = "GAGAL"
                        emoji = "❌"
                    }

                    def message = """
                                    ${emoji} Build Notification
                                    --------------------------------------
                                    Project: ${env.JOB_NAME}
                                    Branch: ${env.BRANCH_NAME}
                                    Commit: ${commitMessage}
                                    SHA: ${env.GIT_COMMIT}
                                    Build: #${env.BUILD_NUMBER}
                                    Status: *${statusMessage}*
                                    --------------------------------------
                                    Check build log: ${env.BUILD_URL}
                                  """.stripIndent()

                    sh """
                        curl -s -X POST https://api.telegram.org/bot\${TELEGRAM_BOT_ID}/sendMessage \\
                        -d chat_id=\${TELEGRAM_CHAT_ID} \\
                        -d message_thread_id=\${TELEGRAM_TOPIC_ID} \\
                        -d parse_mode=Markdown \\
                        --data-urlencode "text=${message}"
                    """
                }
            }
        }
    }
}
```

**Contoh hasil pesan yang dikirim ke Telegram:**
1. Jika build sukses:
![enter image description here](https://i.imgur.com/wXxXmKh_d.webp?maxwidth=760&fidelity=grand)
2. Jika build gagal:
![enter image description here](https://i.imgur.com/RhFDEpv_d.webp?maxwidth=760&fidelity=grand)