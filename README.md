# MY-SUPER-BOT
import telebot
from telebot import types
import yt_dlp
import os
import time
from flask import Flask
from threading import Thread

# ==========================================
# 👇 AAPKI DETAILS (LOCKED)
# ==========================================

# Aapka naya Token
API_TOKEN = '8089344688:AAGyD0Vkr7SM9dUmNQcueEz-sV0gibOcAW0'

# ⚠️ Yahan apni 9-10 digit wali ID likhein (Example: 123456789)
ADMIN_ID = 1234567890 

# ==========================================

bot = telebot.TeleBot(API_TOKEN)
users_db = set()

# --- 24/7 SERVER ---
app = Flask('')

@app.route('/')
def home():
    return "🚀 MASTER BOT IS LIVE! 24/7 ON RENDER"

def run_server():
    app.run(host='0.0.0.0', port=8080)

def keep_alive():
    t = Thread(target=run_server)
    t.start()

# --- FAST DOWNLOADER ---
def download_video_thread(message, url):
    chat_id = message.chat.id
    try:
        msg = bot.send_message(chat_id, "⏳ **Video dhoondh raha hu... Please wait.**")
        
        filename = f"vid_{chat_id}_{int(time.time())}.mp4"
        ydl_opts = {
            'format': 'best[ext=mp4]',
            'outtmpl': filename,
            'quiet': True,
            'no_warnings': True
        }

        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            ydl.download([url])
        
        bot.edit_message_text("✅ **Bas 1 second... Upload ho raha hai!**", chat_id, msg.message_id)
        
        with open(filename, 'rb') as video:
            bot.send_video(chat_id, video, caption=f"Enjoy Boss! 🔥\nDownloaded by @{bot.get_me().username}")
            
        os.remove(filename)
    except Exception as e:
        bot.send_message(chat_id, "❌ **Sorry!** Video download nahi ho saki.\nYa to account private hai ya link galat.")
        if os.path.exists(filename): os.remove(filename)

# --- START & WELCOME ---
@bot.message_handler(commands=['start'])
def send_welcome(message):
    user_id = message.chat.id
    name = message.from_user.first_name
    users_db.add(user_id)

    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    btn1 = types.KeyboardButton("🎮 BGMI Settings")
    btn2 = types.KeyboardButton("🎵 Viral Songs")
    btn3 = types.KeyboardButton("🛠️ Tools Info")
    markup.add(btn1, btn2, btn3)

    text = (
        f"👋 **Namaste {name}!**\n\n"
        "Main aapka Super Fast Bot hu. 🤖\n\n"
        "🎬 **Insta Reel/YouTube Shorts:** Bas link bhejo.\n"
        "👇 **Baaki Settings:** Niche buttons use karo."
    )
    bot.reply_to(message, text, reply_markup=markup)

# --- BROADCAST (Admin Only) ---
@bot.message_handler(commands=['broadcast'])
def broadcast(message):
    if message.chat.id == ADMIN_ID:
        msg = message.text.replace('/broadcast', '')
        if msg == "":
            bot.reply_to(message, "Message likhein: `/broadcast Hello`")
            return
        
        count = 0
        for user in users_db:
            try:
                bot.send_message(user, f"📢 **ADMIN MESSAGE:**\n{msg}")
                count += 1
            except: pass
        bot.reply_to(message, f"✅ {count} logon ko bhej diya.")
    else:
        bot.reply_to(message, "⛔ Access Denied!")

# --- MAIN LOGIC ---
@bot.message_handler(func=lambda message: True)
def handle_all(message):
    text = message.text
    
    if "instagram.com" in text or "youtu" in text:
        Thread(target=download_video_thread, args=(message, text)).start()
    
    elif "BGMI" in text:
        bot.reply_to(message, "🔫 **BGMI Hub:**\n\n🎯 **Sens:** `7052-8888-0000-1111-222`\n🛠️ **Tips:** Close range me Hip-fire use karo.")
    
    elif "Songs" in text:
        bot.reply_to(message, "🎧 **Trending:**\n1. 295\n2. Maan Meri Jaan\n3. Cheques")
        
    elif "Tools" in text:
        bot.reply_to(message, "🛠️ **Tech Tools:**\n- Nmap\n- Zphisher\n- Termux-API")
        
    else:
        if message.chat.id != ADMIN_ID:
            bot.send_message(ADMIN_ID, f"📩 **Msg from {message.from_user.first_name}:**\n{text}")
            bot.reply_to(message, "✅ Aapka message Admin (Boss) ko bhej diya hai.")

# --- RUN ---
if __name__ == "__main__":
    keep_alive()
    bot.infinity_polling()


		
