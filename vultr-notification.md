### Membuat Check Vultr Status dan Notifikasi ke Telegram dengan GitHub Actions
Pada case ini, github actions akan melakukan pengecekan status Vultr pada region tertentu (singapore / dapat diubah), dan mengirimkan notifikasi ke Telegram jika ada alert. Jika tidak ada alert, maka akan mengirimkan pesan "OK" ke Telegram, dengan ketentuan:
1. Menggunakan API Vultr untuk mendapatkan status: `https://status.vultr.com/status.json`
2. Menggunakan bot Telegram untuk mengirimkan notifikasi ke grup Telegram pada topic "status"
3. Menjalankan pengecekan setiap hari pada jam 00:00 SGT (16:00 UTC)
4. Menggunakan github runner `solutionlabs`.

Pada workflow berikut, perlu disiapkan beberapa secret dan variable pada repository github:
- Secrets:
  - `TELEGRAM_BOT_ID`: ID/Token bot Telegram
  - `TELEGRAM_CHAT_ID`: ID chat grup Telegram
  - `TELEGRAM_TOPIC_ID`: ID topic pada grup Telegram
- Variables:
  - `VULTR_REGION`: kode region Vultr, misal `sg` untuk Singapore
  - `API_URL`: URL API Vultr, sekarang digunakan: `https://status.vultr.com/status.json`

Pada workflow berikut, juga terdapat workflow dispatch, sehingga dapat dijalankan secara manual jika diperlukan.

```yml
name: Vultr Status Check

on:
  schedule:
    # 16 utc = 00:00 sgt
    - cron: '0 16 * * *'
  workflow_dispatch:

jobs:
  check-vultr-status:
    runs-on: solutionlabs
    steps:
      - name: Check Vultr Status and Notify
        env:
          TELEGRAM_BOT_ID: ${{ secrets.TELEGRAM_BOT_ID }}
          TELEGRAM_CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
          TELEGRAM_TOPIC_ID: ${{ secrets.TELEGRAM_TOPIC_ID }}
          REGION_CODE: ${{ vars.VULTR_REGION }}
          API_URL: ${{ vars.API_URL }}
        run: |
          echo "Fetching Vultr status region: ${REGION_CODE}"
          ALERTS=$(curl -s --fail "${API_URL}" | jq -e ".regions.${REGION_CODE}.alerts")

          # Cek apakah ada ALERT atau tidak
          if [ -z "$ALERTS" ] || [ "$(echo "$ALERTS" | jq 'length')" -eq 0 ]; then
            #Case 1: AMAN
            echo "No alerts found."
          else

            #Case 2: ADA ALERTS
            echo "Found $(echo "$ALERTS" | jq 'length') alert(s). Sending a notification for each."
            
            #Loop ALERT (mungkin ada lebih dari 1)
            echo "$ALERTS" | jq -c '.[]' | while read -r alert; do
              SUBJECT=$(echo "$alert" | jq -r '.subject')
              CURRENT_STATUS=$(echo "$alert" | jq -r '.status')
              MESSAGE_BODY=$(echo "$alert" | jq -r '.entries[0].message')
              MESSAGE_TEXT=$(printf 'Vultr Status Alert\n\nRegion: %s\nSubject: %s\nStatus: %s\n\n-- Details --\n%s' \
                "$REGION_CODE" "$SUBJECT" "$CURRENT_STATUS" "$MESSAGE_BODY")
              
              API_RESPONSE=$(curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_ID}/sendMessage" \
                --data-urlencode "chat_id=${TELEGRAM_CHAT_ID}" \
                --data-urlencode "message_thread_id=${TELEGRAM_TOPIC_ID}" \
                --data-urlencode "text=${MESSAGE_TEXT}")
              echo "Telegram API Response: $API_RESPONSE"
            done
          fi
```

Sehingga, akan mengirimkan pesan seperti berikut:
1. Jika ada alert:
![enter image description here](https://i.imgur.com/kOMKtAB_d.webp?maxwidth=760&fidelity=grand)
