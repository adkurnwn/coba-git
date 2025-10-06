### Membuat Check Vultr Status dan Notifikasi ke Telegram dengan GitHub Actions
Pada case ini, github actions akan melakukan pengecekan status Vultr pada region tertentu (singapore / dapat diubah), dan mengirimkan notifikasi ke Telegram jika ada alert. dengan ketentuan:
1. Menggunakan API Vultr untuk mendapatkan status: `https://status.vultr.com/status.json`
2. Menggunakan bot Telegram untuk mengirimkan notifikasi ke grup Telegram pada topic "status"
3. Menjalankan pengecekan setiap hari pada jam 00:00 SGT (16:00 UTC)
4. Menggunakan github runner `solutionlabs`.
5. Melakukan mention ke beberapa username di Telegram jika ada alert (opsional, dapat dikosongkan).

Pada workflow berikut, perlu disiapkan beberapa secret dan variable pada repository github:
- Secrets:
  - `TELEGRAM_BOT_ID`: ID/Token bot Telegram
  - `TELEGRAM_CHAT_ID`: ID chat grup Telegram
  - `TELEGRAM_TOPIC_ID`: ID topic pada grup Telegram
- Variables:
  - `VULTR_REGION`: kode region Vultr, misal `sg` untuk Singapore
  - `API_URL`: URL API Vultr, sekarang digunakan: `https://status.vultr.com/status.json`
  - `MENTION_USERNAMES`: username yang akan di-mention jika ada alert, misal `@adkurnwn @example` (opsional, dapat dikosongkan)

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
          MENTION_USERNAMES: ${{ vars.MENTION_USERNAMES }}
        run: |
          sanitize_html() {
            local text="$1"
            text="${text//&/&amp;}"
            text="${text//</&lt;}"
            text="${text//>/&gt;}"
            echo "$text"
          }

          echo "Fetching Vultr status for region: ${REGION_CODE}"
          JSON_RESPONSE=$(curl -s --fail "${API_URL}")

          REGION_NAME=$(echo "$JSON_RESPONSE" | jq -r ".regions.${REGION_CODE}.location")
          ALERTS=$(echo "$JSON_RESPONSE" | jq -e ".regions.${REGION_CODE}.alerts")

          # CASE 1: Tidak ada alert
          if [ -z "$ALERTS" ] || [ "$(echo "$ALERTS" | jq 'length')" -eq 0 ]; then
            echo "No alerts found."
          
          # CASE 2: Ada alert
          else
            echo "Found $(echo "$ALERTS" | jq 'length') alert(s). Sending a notification for each."
            
            CC_LINE=""
            if [ -n "${MENTION_USERNAMES}" ]; then
              CC_LINE="CC: ${MENTION_USERNAMES}"$'\n\n'
            fi
            
            echo "$ALERTS" | jq -c '.[]' | while read -r alert; do
              SUBJECT=$(echo "$alert" | jq -r '.subject')
              CURRENT_STATUS=$(echo "$alert" | jq -r '.status')
              MESSAGE_BODY=$(echo "$alert" | jq -r '.entries[0].message')
              
              SAFE_SUBJECT=$(sanitize_html "$SUBJECT")
              SAFE_MESSAGE_BODY=$(sanitize_html "$MESSAGE_BODY")
              SAFE_REGION_NAME=$(sanitize_html "$REGION_NAME")
              
              MESSAGE_TEXT=$(printf '<b>Vultr Status Alert (%s)</b>\n\n%s<b>Subject:</b> %s\n<b>Status:</b> %s\n\n<b>Details:</b>\n<pre>%s</pre>' \
                "$SAFE_REGION_NAME" "$CC_LINE" "$SAFE_SUBJECT" "$CURRENT_STATUS" "$SAFE_MESSAGE_BODY")
              
              API_RESPONSE=$(curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_ID}/sendMessage" \
                --data-urlencode "chat_id=${TELEGRAM_CHAT_ID}" \
                --data-urlencode "message_thread_id=${TELEGRAM_TOPIC_ID}" \
                --data-urlencode "text=${MESSAGE_TEXT}" \
                --data-urlencode "parse_mode=HTML")
              echo "Telegram API Response: $API_RESPONSE"
            done
          fi
```
Pada script pada workflow tersebut, ditambahkan fungsi `sanitize_html` untuk menghindari dan mengubah karakter khusus HTML pada pesan yang dikirim ke Telegram. Sehingga, pesan yang dikirim dapat ditampilkan dengan benar di Telegram.

misalnya response API Status Vultr berikut yang memiliki karakter khusus HTML seperti `\r\n\r\n`
```json
.....
    "cdg": {
      "location": "Paris",
      "country": "FR",
      "country_name": "France",
      "alerts": [
        {
          "id": "7f4f4e29-f65e-4933-87c5-110f10b63775",
          "subject": "Paris Scheduled Maintenance - 2025-10-07",
          "status": "ongoing",
          "start_date": "2025-09-29T20:25:00+00:00",
          "updated_at": "",
          "entries": [
            {
              "updated_at": "2025-09-29T20:25:00+00:00",
              "message": "Event Type: Network Upgrade\r\n\r\nWe are performing system changes in the Paris location during the following scheduled maintenance window. \r\n\r\nStart Time: 2025-10-07 22:00:00 UTC\r\nEnd Time: 2025-10-08 02:00:00 UTC\r\n\r\nWe schedule higher impact maintenance events during off-peak times to reduce impact to customer infrastructure. Our engineers make every effort to minimize system impact, however, network reachability to Paris instances may be affected for some, or all, of the scheduled maintenance window as we perform this work. While it’s important to be aware of start and end times of the maintenance window, due to our redundant network architecture, our engineers expect that you may see little to no impact at all while the work is performed."
            }
          ]
        }
      ]
    },
  .....
```

Sehingga, workflow tersebut akan mengirimkan pesan seperti berikut:
1. Jika ada alert:
![enter image description here](https://i.imgur.com/N10NcCX_d.webp?maxwidth=760&fidelity=grand)
