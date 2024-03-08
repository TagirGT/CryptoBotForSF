# CryptoBotForSF


import telebot
from currency_converter import CurrencyConverter
from telebot import types

bot = telebot.TeleBot("7190453683:AAE4Inp2JiHoXzfVCu-GYmK5h6zZIK3kb0I")
currency = CurrencyConverter()
amount = 0


@bot.message_handler(commands=["start"])
def echo_test(message):
    bot.send_message(message.chat.id,"Привет, введите сумму")
    bot.register_next_step_handler(message, summa)

def summa(message):
    global amount
    try:
        amount = int(message.text.strip())
    except ValueError:
        bot.send_message(message.chat.id, "Неверный формат. Впишите сумму")
        bot.register_next_step_handler(message, summa)
        return

    if amount > 0:
        markup = types.InlineKeyboardMarkup(row_width=2)
        btn1 = types.InlineKeyboardButton("USD,EUR", callback_data="usd/eur")
        btn2 = types.InlineKeyboardButton("EUR,USD", callback_data="eur/usd")
        btn3 = types.InlineKeyboardButton("RUB,USD", callback_data="rub/usd")
        btn4 = types.InlineKeyboardButton("Others", callback_data="else" )
        markup.add(btn1,btn2,btn3,btn4)
        bot.send_message(message.chat.id, "Выберите пару валют", reply_markup=markup)
    else:
        bot.send_message(message.chat.id, "Число должно быть больше за 0. Впишите сумму.")
        bot.register_next_step_handler(message, summa)


@bot.callback_query_handler(func=lambda call: True)
def callback(call):
    if call.data != "else":
        value = call.data.upper().split("/")
        res = currency.convert(amount, value[0], value[1])
        bot.send_message(call.message.chat.id, f"Получается: {round(res, 2)}. Можете заново ввести сумму.")
        bot.register_next_step_handler(call.message, summa)
    else:
        bot.send_message(call.message.chat.id, "Введите пару значений через слэш")
        bot.register_next_step_handler(call.message, my_currency)

def my_currency(message):
    try:
        value = message.text.upper().split("/")
        res = currency.convert(amount, value[0], value[1])
        bot.send_message(message.chat.id, f"Получается: {round(res, 2)}. Можете заново ввести сумму.")
        bot.register_next_step_handler(message, summa)
    except Exception:
        bot.send_message(message.chat.id, "Что-то не так. Введите сумму.")
        bot.register_next_step_handler(message, summa)

@bot.message_handler(commands=["help"])
def echo_test(message):
    bot.send_message(message.chat.id,"Привет, чтобы просмотреть информацию о достпуной валюте впишите /value ")

@bot.message_handler(commands=["value"])
def echo_test(message):
    bot.send_message(message.chat.id, "USD - Доллар///EUR - Евро///RUB - Рубль")

bot.polling(none_stop=True)
