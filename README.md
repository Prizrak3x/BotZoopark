
import telebot
from telebot import types

TOKEN = ''  # Замените на токен вашего бота
bot = telebot.TeleBot(TOKEN)

# Вопросы и ответы для викторины
QUIZ = [
    {
        'question': 'Какое животное может видеть и цвета, и ультрафиолетовый свет?',
        'options': ['Пчела', 'Слепой крот'],
        'correct': 0,
        'animal': 'Золотистый Тамарин',
        'image_url': 'https://moscowzoo.ru/upload/iblock/39e/39e56a364f4278b167d93d8b94d75510.jpg',
        'description': 'В природе этих чудных обезьянок осталось немногим более 3000, ещё около 500 содержатся в зоопарках мира. К сожалению, численность их в природе продолжает снижаться.'
    },
    {
        'question': 'Какое животное всегда спит с открытыми глазами?',
        'options': ['Коала', 'Золотая рыбка'],
        'correct': 1,
        'animal': 'Рыбка Телескоп',
        'image_url': 'https://krasivosti.pro/uploads/posts/2021-04/1618484286_17-krasivosti_pro-p-riba-s-vipuklimi-glazami-ribi-krasivo-foto-18.jpg',
        'description': 'Телескоп (в азиатских странах носит название «дэмэкин») – короткотелая селекционная форма золотой рыбки, отличительной особенностью которой являются огромные выпуклые глаза по бокам головы'
    },
    # Добавьте больше вопросов здесь...
]

# Индекс текущего вопроса
current_question = 0

# Счетчик правильных ответов
correct_answers = 0

# Описание викторины
quiz_description = ''

@bot.message_handler(commands=['start'])
def send_welcome(message):
    markup = types.ReplyKeyboardMarkup(row_width=1)
    itembtn1 = types.KeyboardButton('/quiz')
    itembtn2 = types.KeyboardButton('Узнать о программе опеки')  # Добавляем кнопку здесь
    markup.add(itembtn1, itembtn2)
    bot.send_message(message.chat.id, "Привет! Готовы начать викторину? Нажмите кнопку /quiz ниже, чтобы начать. Или нажмите 'Узнать о программе опеки', чтобы узнать больше о программе опеки над животными.", reply_markup=markup)

@bot.message_handler(func=lambda message: message.text == 'Узнать о программе опеки')
def show_care_program(message):

    care_program_info = '''Стать опекуном
Участие в программе «Возьми животное под опеку» — это ваш личный вклад в дело сохранения биоразнообразия Земли и развитие нашего зоопарка.

Основная задача Московского зоопарка с самого начала его существования это — сохранение биоразнообразия нашей планеты. Когда вы берете под опеку животное, вы помогаете нам в этом благородном деле.

При нынешних темпах развития цивилизации к 2050 году с лица Земли могут исчезнуть около 10 000 биологических видов. Московский зоопарк вместе с другими зоопарками мира делает все возможное, чтобы сохранить их.

Опека — это прекрасная возможность принять участие в деле сохранения редких видов, помочь нам в реализации природоохранных программ.

Программа «Возьми животное под опеку» дает возможность опекунам ощутить свою причастность к делу сохранения природы, участвовать в жизни Московского зоопарка и его обитателей, видеть конкретные результаты своей деятельности. Опекать – значит помогать любимым животным.

В настоящее время опекуны объединились в неформальное сообщество — Клуб друзей Московского зоопарка.

5 уровней участия в программе:
 1. Индивидуальный (пожертвование до 50 тыс. рублей в год)
 2. Партнерский (пожертвование от 50 до 150 тыс. рублей в год)
 3. Представительский (пожертвование от 150 до 300 тыс. рублей в год)
 4. Почетный (пожертвование от 300 тыс. до 1 млн. рублей в год)
 5. Президентский (пожертвование от 1 млн. рублей в год и более)

Как выбрать животное?

Сейчас в Московском зоопарке содержится около 6 000 особей, относящихся примерно к 1 100 биологическим видам мировой фауны. Каждое животное уникально и неповторимо. Взять под опеку можно любое животное, которое вам нравится и соответствует вашим финансовым возможностям.

Стоимость опеки рассчитывается из ежедневного рациона питания животного.

Если вы уже выбрали животное и хотите узнать стоимость опеки над ним, вам нужно отправить свой запрос на почту или позвонить по номеру.

Контакты для связи:
 
+7(962) 971 38 75 c 9:00 до 18:00

zoofriends@moscowzoo.ru'''
    bot.send_message(message.chat.id, care_program_info)

def ask_question(message):
    global current_question
    question = QUIZ[current_question]['question']
    options = QUIZ[current_question]['options']
    markup = types.ReplyKeyboardMarkup(row_width=2)
    itembtns = [types.KeyboardButton(option) for option in options]
    markup.add(*itembtns)
    bot.send_message(message.chat.id, question, reply_markup=markup)

@bot.message_handler(commands=['quiz'])
def start_quiz(message):
    global current_question, correct_answers
    current_question = 0
    correct_answers = 0
    ask_question(message)

@bot.message_handler(func=lambda message: True)
def check_answer(message):
    global current_question, correct_answers
    # Проверяем, что сообщение является ответом на вопрос
    if current_question < len(QUIZ) and message.text in QUIZ[current_question]['options']:
        selected_option = message.text
        if selected_option == QUIZ[current_question]['options'][QUIZ[current_question]['correct']]:
            correct_answers += 1
        # Переходим к следующему вопросу
        current_question += 1
        if current_question < len(QUIZ):
            ask_question(message)
        else:
            end_quiz(message)

def end_quiz(message):
    global quiz_description
    if correct_answers > len(QUIZ) / 2:
        animal = 'Золотистый Тамарин'
        image_url = 'https://moscowzoo.ru/upload/iblock/39e/39e56a364f4278b167d93d8b94d75510.jpg'
        quiz_description = 'В природе этих чудных обезьянок осталось немногим более 3000, ещё около 500 содержатся в зоопарках мира. К сожалению, численность их в природе продолжает снижаться.'
    else:
        animal = 'Рыбка Телескоп'
        image_url = 'https://krasivosti.pro/uploads/posts/2021-04/1618484286_17-krasivosti_pro-p-riba-s-vipuklimi-glazami-ribi-krasivo-foto-18.jpg'
        quiz_description = 'Телескоп (в азиатских странах носит название «дэмэкин») – короткотелая селекционная форма золотой рыбки, отличительной особенностью которой являются огромные выпуклые глаза по бокам головы'
    text = f'Ваше тотемное животное - {animal}.'
    bot.send_photo(message.chat.id, image_url, caption=text)
    send_quiz_description(message)

def send_quiz_description(message):
    markup_inline = types.InlineKeyboardMarkup()
    itembtn_inline = types.InlineKeyboardButton('Описание животного', callback_data='description')
    markup_inline.add(itembtn_inline)
    bot.send_message(message.chat.id, "Нажмите кнопку ниже, чтобы узнать больше о вашем тотемном животном.", reply_markup=markup_inline)

    markup_reply = types.ReplyKeyboardRemove()
    bot.send_message(message.chat.id, "Для повторения викторины введите команду /start", reply_markup=markup_reply)

@bot.callback_query_handler(func=lambda call: call.data == 'description')
def show_description(call):
    bot.answer_callback_query(call.id)
    bot.send_message(call.message.chat.id, quiz_description)



bot.polling()




