import requests
from bs4 import BeautifulSoup
from telegram import Bot
import schedule
import time

BOT_TOKEN ="7678395014:AAE0kJkITgduV0JOn6sIOTus9zBjNeR26rs"
CHAT_ID = "394970885"
bot = Bot(BOT_TOKEN)

# Keywords to track
KEYWORDS = ["FY1", "FY2", "CT1", "Junior Clinical Fellow"]

# List of job sources
URLS = {
    "NHS Jobs": "https://www.jobs.nhs.uk/candidate/search",
    "NHS Scotland": "https://apply.jobs.scot.nhs.uk/vacancies.aspx?txtKeyword=junior",
    "HSCNI": "https://jobs.hscni.net/Job/Search?Keywords=junior",
    "HealthJobsUK": "https://www.healthjobsuk.com/job_search?search=junior+clinical+fellow"
}

# Store seen job URLs
seen = set()

def check_jobs():
    for site, url in URLS.items():
        try:
            response = requests.get(url, timeout=10)
            soup = BeautifulSoup(response.text, 'html.parser')
            links = soup.find_all('a', href=True)

            for link in links:
                href = link['href']
                text = link.get_text(strip=True)

                if any(kw.lower() in text.lower() for kw in KEYWORDS):
                    full_url = href if href.startswith("http") else url.split("/")[2].split("?")[0] + href
                    if full_url not in seen:
                        seen.add(full_url)
                        bot.send_message(chat_id=CHAT_ID, text=f"🩺 *{site}*:\n{text}\n🔗 {full_url}", parse_mode="Markdown")
        except Exception as e:
            print(f"Error on {site}: {e}")

# Schedule to run every 3 mins
schedule.every(3).minutes.do(check_jobs)

print("Bot started... Press Ctrl+C to stop.")
check_jobs()  # Initial run

while True:
    schedule.run_pending()
    time.sleep(1)
