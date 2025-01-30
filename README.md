# 1hnews
 a simple news bot for telegramm that shows the most current cryptocurrency news with updated information every hour
import asyncio
import logging
import aiohttp
import ssl
from aiogram import Bot, Dispatcher, ParseMode, types
from bs4 import BeautifulSoup

TOKEN = "YOUR_TELEGRAM_BOT_TOKEN"
NEWS_URL = "https://cointelegraph.com/"

bot = Bot(token=TOKEN)
dp = Dispatcher(bot)

async def fetch_news():
    ssl_context = ssl.create_default_context()
    async with aiohttp.ClientSession(connector=aiohttp.TCPConnector(ssl=ssl_context)) as session:
        async with session.get(NEWS_URL) as response:
            page_content = await response.text()
            soup = BeautifulSoup(page_content, 'html.parser')
            articles = soup.find_all('article', limit=5)
            news_list = []
            for article in articles:
                title = article.find('a', class_='post-card-inline__title-link').text.strip()
                link = "https://cointelegraph.com" + article.find('a', class_='post-card-inline__title-link')['href']
                news_list.append(f"<b>{title}</b>\n<a href='{link}'>Читать подробнее</a>")
            return news_list

@dp.message_handler(commands=['news'])
async def send_news(message: types.Message):
    news = await fetch_news()
    if news:
        await message.answer("\n\n".join(news), parse_mode=ParseMode.HTML, disable_web_page_preview=True)
    else:
        await message.answer("Не удалось получить новости. Попробуйте позже.")

async def scheduled_news():
    while True:
        news = await fetch_news()
        if news:
            await bot.send_message("@your_channel_or_user_id", "\n\n".join(news), parse_mode=ParseMode.HTML, disable_web_page_preview=True)
        await asyncio.sleep(3600)  # Обновление раз в час

async def main():
    logging.basicConfig(level=logging.INFO)
    loop = asyncio.get_event_loop()
    loop.create_task(scheduled_news())
    await dp.start_polling()

if __name__ == "__main__":
    asyncio.run(main())
