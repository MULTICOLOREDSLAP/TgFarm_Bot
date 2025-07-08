# TgFarm_Bot

import telebot
import json

bot = telebot.TeleBot('7947379155:AAGKNtj5RG9uk98BHcdWCTUyU41Ftrbi13Y')

PROMOCODES = {
    "WELCOMETOBOT": {"coins": 55, "crystals": 5, "limit": None},
    "2025YEAR": {"coins": 350, "crystals": 30, "limit": 25},
    "2025SUMMER": {"coins": 100, "crystals": 7, "limit": 92},
    "2025JULY": {"coins": 200, "crystals": 10, "limit": 31},
    "RLPSFVTJQWVMB": {"coins": 1000, "crystals": 100, "limit": 5}
}

users = {}

def load_data():
    global users
    try:
        with open('users.json', 'r') as f:
            users = json.load(f)
    except:
        users = {}

def save_data():
    with open('users.json', 'w') as f:
        json.dump(users, f)

load_data()

def get_user(user_id):
    if user_id not in users:
        users[user_id] = {
            "balance": 100,
            "crystals": 0,
            "carrot": 1,
            "cabbage": 0,
            "cow": 0,
            "chicken": 0,
            "milk": 0,
            "eggs": 0,
            "vip_level": 0,
            "promo": []
        }
        save_data()
    return users[user_id]

@bot.message_handler(commands=['start'])
def start(message):
    user_id = str(message.from_user.id)
    get_user(user_id)
    bot.send_message(message.chat.id, "🌱 Добро пожаловать в ФЕРМУ! Вы получили 1 🥕 и 100 монет.")

@bot.message_handler(commands=['profile'])
def profile(message):
    user_id = str(message.from_user.id)
    user = get_user(user_id)
    text = f"""👤 Профиль:
💰 Монеты: {user['balance']}
💎 Кристаллы: {user['crystals']}
🥕 Морковка: {user['carrot']}
🥬 Капуста: {user['cabbage']}
🐄 Коровы: {user['cow']}
🐓 Курицы: {user['chicken']}
🥛 Молоко: {user['milk']}
🥚 Яйца: {user['eggs']}
⭐ VIP: {user['vip_level']}
Промокоды: {', '.join(user['promo'])}
"""
    bot.send_message(message.chat.id, text)

@bot.message_handler(commands=['ActivitovatPromo'])
def activate_promo(message):
    user_id = str(message.from_user.id)
    user = get_user(user_id)
    args = message.text.split()
    if len(args) < 2:
        bot.send_message(message.chat.id, "❌ Введите промокод! Например: /ActivitovatPromo WELCOMETOBOT")
        return
    promo = args[1].strip().upper()
    if promo not in PROMOCODES:
        bot.send_message(message.chat.id, "❌ Промокод не найден.")
        return
    if promo in user['promo']:
        bot.send_message(message.chat.id, "❌ Вы уже активировали этот промокод.")
        return
    user['balance'] += PROMOCODES[promo]['coins']
    user['crystals'] += PROMOCODES[promo]['crystals']
    user['promo'].append(promo)
    save_data()
    bot.send_message(message.chat.id, f"✅ Промокод активирован! +{PROMOCODES[promo]['coins']} монет и +{PROMOCODES[promo]['crystals']} кристаллов.")

@bot.message_handler(commands=['plant'])
def plant(message):
    user_id = str(message.from_user.id)
    user = get_user(user_id)
    user['carrot'] += 1
    save_data()
    bot.send_message(message.chat.id, "🌱 Вы посадили морковку! Через час вы сможете собрать урожай.")

@bot.message_handler(commands=['harvest'])
def harvest(message):
    user_id = str(message.from_user.id)
    user = get_user(user_id)
    if user['carrot'] > 0:
        coins_earned = 50 * user['carrot']
        user['balance'] += coins_earned
        user['carrot'] = 0
        save_data()
        bot.send_message(message.chat.id, f"✅ Урожай собран! Вы получили {coins_earned} монет.")
    else:
        bot.send_message(message.chat.id, "❌ У вас нет посаженной морковки.")

@bot.message_handler(commands=['buy_animal'])
def buy_animal(message):
    user_id = str(message.from_user.id)
    user = get_user(user_id)
    text = "Напишите, кого купить: cow (200 монет) или chicken (100 монет)"
    bot.send_message(message.chat.id, text)

@bot.message_handler(func=lambda m: m.text.lower() in ['cow', 'chicken'])
def buy_animal_choice(message):
    user_id = str(message.from_user.id)
    user = get_user(user_id)
    choice = message.text.lower()
    if choice == 'cow':
        if user['balance'] >= 200:
            user['balance'] -= 200
            user['cow'] += 1
            bot.send_message(message.chat.id, "🐄 Вы купили корову!")
        else:
            bot.send_message(message.chat.id, "❌ Недостаточно монет.")
    elif choice == 'chicken':
        if user['balance'] >= 100:
            user['balance'] -= 100
            user['chicken'] += 1
            bot.send_message(message.chat.id, "🐓 Вы купили курицу!")
        else:
            bot.send_message(message.chat.id, "❌ Недостаточно монет.")
    save_data()

@bot.message_handler(commands=['sell'])
def sell(message):
    user_id = str(message.from_user.id)
    user = get_user(user_id)
    milk_income = user['cow'] * 50
    egg_income = user['chicken'] * 20
    user['balance'] += milk_income + egg_income
    bot.send_message(message.chat.id, f"🥛 Продано молоко: +{milk_income} монет\n🥚 Продано яйца: +{egg_income} монет")
    save_data()

@bot.message_handler(commands=['vip'])
def vip(message):
    bot.send_message(message.chat.id, "⭐ Vip можно купить через /market или получить от админов!")

@bot.message_handler(commands=['market'])
def market(message):
    bot.send_message(message.chat.id, "🛒 Рынок пока в разработке!")

@bot.message_handler(commands=['trade'])
def trade(message):
    bot.send_message(message.chat.id, "🔁 Трейд с друзьями будет реализован позже!")

@bot.message_handler(commands=['help'])
def help(message):
    text = """
📜 Доступные команды:
/start - начать игру
/profile - ваш профиль
/ActivitovatPromo [код] - активировать промокод
/plant - посадить морковку
/harvest - собрать урожай
/buy_animal - купить животное
/sell - продать молоко и яйца
/vip - информация о VIP
/market - рынок
/trade - трейды
/help - помощь
"""
    bot.send_message(message.chat.id, text)

bot.polling()
