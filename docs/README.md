# Полное техническое руководство по созданию чат-бота в Telegram для обучения правильному ударению в словах
## Введение
Созданный чат-бот - это чат с тестом для изучения и запоминания правильного ударения

  **Технологический стек:**
  
  Библиотека telegram на python
  
  **Среда разработки:**
  
  PyCharm

## Настройка среды разработки
  1. Скачать PyCharm
  
  2. В проекте написать: **pip install telegram** для установки всех нужных библиотек
## Работа бота
![alt text](https://github.com/ctrannik787/PD/blob/main/start.png?raw=true)
### 1. Реализация команд

```python
async def start(update: Update, context):
    message_text = (
        "Здравствуйте! Этот бот поможет вам проверить правильное ударение в словах.\n"
        "Доступные команды:\n"
        "/quiz — начать тест\n"
        "Чтобы начать, введите /quiz"
    )
    if hasattr(update, 'message') and update.message:
        await update.message.reply_text(message_text)
    elif hasattr(update, 'callback_query') and update.callback_query:
        await update.callback_query.edit_message_text(message_text)

async def quiz(update: Update, context):
    # Инициализация данных для пользователя
    context.user_data['score'] = 0
    context.user_data['question_indices'] = list(range(len(QUESTIONS)))
    random.shuffle(context.user_data['question_indices'])
    await send_question(update, context)
```
### 2. Создание вопроса
![alt text](https://github.com/ctrannik787/PD/blob/main/question.png?raw=true)
```python
async def send_question(update: Update, context):
    """Отправка вопроса или финального результата."""
question_idx = context.user_data['question_indices'].pop()
    context.user_data['current_question'] = question_idx
    question_word = QUESTIONS[question_idx]
    correct_answer = question_word

    # собираем неправильные варианты, исключая правильный
    incorrects_pool = [opt for opt in INCORRECT_OPTIONS if opt.lower() != correct_answer.lower()]

    selected_wrong_options = []
    used_words = {correct_answer.lower()}
    attempts = 0
    max_attempts = 100

    while len(selected_wrong_options) < 3 and attempts < max_attempts:
        candidate = random.choice(incorrects_pool)
        candidate_word = candidate.lower()
        if candidate_word not in used_words:
            selected_wrong_options.append(candidate)
            used_words.add(candidate_word)
        attempts += 1

    # если не удалось подобрать 3 уникальных варианта
    if len(selected_wrong_options) < 3:
        remaining_needed = 3 - len(selected_wrong_options)
        remaining_options = [opt for opt in incorrects_pool if opt not in selected_wrong_options]
        selected_wrong_options.extend(random.sample(remaining_options, remaining_needed))

    options = selected_wrong_options + [correct_answer]
    random.shuffle(options)
```
### 3. Отправка вопроса
```python
 # создаем кнопки
    keyboard = [[InlineKeyboardButton(opt, callback_data=opt)] for opt in options]
    reply_markup = InlineKeyboardMarkup(keyboard)

    # отправляем сообщение
    if hasattr(update, 'callback_query') and update.callback_query:
        await update.callback_query.edit_message_text(
            text="Выберите правильное ударение в слове:",
            reply_markup=reply_markup
        )
    elif hasattr(update, 'message') and update.message:
        await update.message.reply_text(
            text="Выберите правильное ударение:",
            reply_markup=reply_markup
        )
```
### 4. Обработка ответа
```python
    if selected == correct_answer:
        # правильный ответ
        context.user_data['score'] = context.user_data.get('score', 0) + 1
        await send_question(update, context)
    else:
        # неправильный — завершение теста
        score = context.user_data.get('score', 0)
        text = (
            f"Неправильно! Тест завершен.\n\n"
            f"Правильных ответов: {score}.\n"
            f"Правильный ответ: '{QUESTIONS[current_idx]}'. Ваш ответ: '{selected}'."
        )
        # добавим кнопку "Начать заново"
        restart_button = InlineKeyboardButton("Начать заново", callback_data='restart')
        reply_markup = InlineKeyboardMarkup([[restart_button]])
        await query.edit_message_text(text, reply_markup=reply_markup)
```
## Заключение
Это руководство охватывает ключевые аспекты создания чат-бота в telegram на Python. Для дальнейшего развития проекта стоит рассмотреть:
1. Добавление уровней сложности
2. Сохранение лучшего результата
