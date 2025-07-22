from aiogram import Bot, Dispatcher, executor, types

API_TOKEN = '7883302262:AAFty9F2q3noH_-0Vujp7NMOt-6jamLexJ0'

bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

registrations = {}

@dp.message_handler(commands=['start'])
async def start(message: types.Message):
    await message.answer("Привет! Используй /registration чтобы войти или выйти из списка.")

@dp.message_handler(commands=['registration'])
async def registration(message: types.Message):
    keyboard = types.InlineKeyboardMarkup(row_width=2)
    keyboard.add(
        types.InlineKeyboardButton(text="👍🏻 Войти в список", callback_data="join"),
        types.InlineKeyboardButton(text="👎🏻 Выйти из списка", callback_data="leave")
    )
    sent = await message.answer("Выберите действие:", reply_markup=keyboard)
    registrations[sent.message_id] = []

@dp.callback_query_handler(lambda c: c.data in ['join', 'leave'])
async def handle_registration(callback_query: types.CallbackQuery):
    user = callback_query.from_user
    message_id = callback_query.message.message_id
    user_list = registrations.get(message_id, [])

    if callback_query.data == 'join':
        if user.id in user_list:
            await callback_query.answer("Вы уже в списке.")
        else:
            user_list.append(user.id)
            registrations[message_id] = user_list
            await callback_query.answer("Вы добавлены в список.")
    else:  # leave
        if user.id in user_list:
            user_list.remove(user.id)
            registrations[message_id] = user_list
            await callback_query.answer("Вы удалены из списка.")
        else:
            await callback_query.answer("Вас нет в списке.")

    names = []
    for uid in user_list:
        member = await bot.get_chat_member(callback_query.message.chat.id, uid)
        names.append(f"@{member.user.username}" if member.user.username else member.user.full_name)

    updated_text = f"📋 Зарегистрированы ({len(user_list)}):\n" + "\n".join(names) if names else "📋 Список пуст."
    await callback_query.message.edit_text(updated_text, reply_markup=callback_query.message.reply_markup)

if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
