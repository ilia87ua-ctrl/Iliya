# Iliya
# main.py
# ربات سریع تبدیل ویس به متن - بدون نیاز به ffmpeg
import os
from aiogram import Bot, Dispatcher, types
from aiogram.utils import executor
from faster_whisper import WhisperModel
import tempfile
from pathlib import Path

BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
bot = Bot(token=BOT_TOKEN)
dp = Dispatcher(bot)

# مدل کوچک و سریع (رایگان)
model = WhisperModel("small")

@dp.message_handler(content_types=[types.ContentType.VOICE])
async def handle_voice(message: types.Message):
    with tempfile.TemporaryDirectory() as td:
        input_file = Path(td)/"voice.ogg"
        await message.voice.download(destination_file=input_file)
        segments, _ = model.transcribe(str(input_file))
        text = " ".join(seg.text for seg in segments)
    await message.reply(text or "(متنی تشخیص داده نشد)")

@dp.message_handler(commands=["start"])
async def start_cmd(message: types.Message):
    await message.reply("سلام! ویس بفرست تا متنش رو برگردونم.")

if __name__ == "__main__":
    executor.start_polling(dp)
