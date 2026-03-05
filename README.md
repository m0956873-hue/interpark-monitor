# interpark-monitor
import requests
import time
from bs4 import BeautifulSoup

# Interpark schedule頁面
URL = "https://tickets.interpark.com/onestop/schedule"

# 把這裡改成你的Discord webhook
WEBHOOK_URL = "https://discord.com/api/webhooks/1479076004927508511/qADXs4qBGVQmzIEfKQ9QZazSZ8myEVH0MGsWFum0SNuwYkXUOjCB3dVI6B-FR55DnWIQ"

# 已經看過的演出
seen_events = set()


def send_discord(message):
    data = {
        "content": message
    }
    requests.post(WEBHOOK_URL, json=data)


def check_interpark():

    global seen_events

    headers = {
        "User-Agent": "Mozilla/5.0"
    }

    r = requests.get(URL, headers=headers)

    soup = BeautifulSoup(r.text, "html.parser")

    links = soup.find_all("a")

    new_events = []

    for link in links:

        name = link.get_text(strip=True)

        if len(name) > 5:

            if name not in seen_events:

                seen_events.add(name)
                new_events.append(name)

    return new_events


print("Interpark monitor started")


while True:

    try:

        events = check_interpark()

        for e in events:

            msg = f"🎫 Interpark 新演出發現！\n{e}\n{URL}"

            send_discord(msg)

            print("found:", e)

    except Exception as err:

        print("error:", err)

    time.sleep(60)
