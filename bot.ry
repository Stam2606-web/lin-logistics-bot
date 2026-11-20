import os
import random
import threading
from flask import Flask
from telegram import Update, KeyboardButton, ReplyKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

app_flask = Flask(__name__)
@app_flask.route('/')
def home():
    return "Bot is running!"

def run_flask():
    port = int(os.environ.get("PORT", 10000))
    app_flask.run(host="0.0.0.0", port=port)

clients = {}

def generate_code():
    return f"LIN-{random.randint(1000, 9999)}"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Привет! Это Lin Logistics. Как вас зовут?")
    context.user_data['step'] = 'name'

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    step = context.user_data.get('step')
    text = update.message.text
    if step == 'name':
        context.user_data['name'] = text
        context.user_data['step'] = 'phone'
        button = KeyboardButton("Отправить номер", request_contact=True)
        markup = ReplyKeyboardMarkup([[button]], resize_keyboard=True, one_time_keyboard=True)
        await update.message.reply_text("Спасибо! Теперь отправьте номер кнопкой ниже:", reply_markup=markup)
    elif step == 'phone':
        phone = update.message.contact.phone_number if update.message.contact else text
        name = context.user_data.get('name', 'Клиент')
        if user_id in clients:
            code = clients[user_id]['code']
        else:
            code = generate_code()
            clients[user_id] = {'name': name, 'phone': phone, 'code': code}
        await update.message.reply_text(f"✅ Вы зарегистрированы в Lin Logistics!\n\n👤 Имя: {name}\n📱 Телефон: {phone}\n🔑 Ваш личный код: *{code}*\n\nУказывайте этот код при заказе.", parse_mode='Markdown')
        context.user_data.clear()

def main():
    TOKEN = os.environ.get("TOKEN")
    threading.Thread(target=run_flask, daemon=True).start()
    app = Application.builder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT | filters.CONTACT, handle_message))
    app.run_polling()

if __name__ == "__main__":
    main()
