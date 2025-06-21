# File-sharing-bot-1

from pyrogram import Client, filters
from pyrogram.types import Message
import asyncio
import os

# ========== CONFIG ==========
API_ID = 11756460
API_HASH = "fd24d30016b3bd343f635609e71020b5"
BOT_TOKEN = "7872420234:AAF-4WJ_Jn_h7taBReGYwMFFhTD7hNf4cRc"
OWNER_ID = 2002919767
ADMIN_IDS = [2002919767]
FORCE_SUB = "@Jax_Movies"
CHANNEL_ID = -1001573789708
BOT_USERNAME = "New Movies Bot"
AUTO_DELETE_TIME = 300 in seconds
START_PIC_PATH = "https://i.ibb.co/zTLS8ms4/image.jpg"

# ========== INIT BOT ==========
bot = Client("file_share_bot", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN)


# ========== UTILITY FUNCTIONS ==========
async def is_subscribed(user_id):
    try:
        user = await bot.get_chat_member(FORCE_SUB, user_id)
        return user.status in ("member", "administrator", "creator")
    except:
        return False


# ========== COMMAND HANDLERS ==========
@bot.on_message(filters.command("start") & filters.private)
async def start_command(client, message: Message):
    if not await is_subscribed(message.from_user.id):
        await message.reply_text(f"📢 Please join {FORCE_SUB} to use this bot.")
        return

    if "dl_" in message.text:
        try:
            file_id = int(message.text.split("dl_")[1])
            await bot.copy_message(
                chat_id=message.chat.id,
                from_chat_id=CHANNEL_ID,
                message_id=file_id
            )
        except:
            await message.reply_text("❌ File not found or expired.")
    else:
        if os.path.exists(START_PIC_PATH):
            await message.reply_photo(START_PIC_PATH, caption="👋 Welcome! Send me any file and I’ll give you a shareable link.")
        else:
            await message.reply_text("👋 Welcome! Send me any file and I’ll give you a shareable link.")


@bot.on_message(filters.private & filters.media)
async def handle_file(client, message: Message):
    if not await is_subscribed(message.from_user.id):
        await message.reply_text(f"📢 Join {FORCE_SUB} to use this bot.")
        return

    try:
        sent = await message.copy(chat_id=CHANNEL_ID)
        file_id = sent.id
        link = f"https://t.me/{BOT_USERNAME}?start=dl_{file_id}"
        await message.reply_text(f"✅ File saved!\n🔗 Share link:\n{link}\n🕒 This file will auto-delete in {AUTO_DELETE_TIME//60} minutes.")

        # Schedule deletion
        await asyncio.sleep(AUTO_DELETE_TIME)
        await bot.delete_messages(CHANNEL_ID, file_id)

    except Exception as e:
        await message.reply_text(f"❌ Error saving file: {e}")


@bot.on_message(filters.command("broadcast") & filters.user(ADMIN_IDS))
async def broadcast(client, message: Message):
    await message.reply_text("🛠 Broadcast feature is not implemented in this version.")


# ========== START BOT ==========
print("🚀 Bot is running...")
bot.run()
