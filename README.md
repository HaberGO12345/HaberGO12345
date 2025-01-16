from telegram import Update, ReplyKeyboardMarkup, ReplyKeyboardRemove
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes
from PIL import Image
from io import BytesIO

# Токен бота
TOKEN = '8126071025:AAFCsE71KnCGdsOTCliYDXslY86fKOCRhBQ'

# Меню кнопок
main_menu = ReplyKeyboardMarkup(
    [['👁 Інструкція', 'ℹ️ Про бота', '📸 Відправити Фото']],
    resize_keyboard=True
)

# Команда /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_first_name = update.effective_user.first_name  # Ім'я користувача
    await update.message.reply_text("Ласкаво просимо!", reply_markup=main_menu)

    # Друге повідомлення
    welcome_message = (
        f"👋 Привіт, {user_first_name}!\n"
        "Мене звуть RedPhoto, я бот-нейромережа, який допоможе тобі!✨\n\n"
        "👁 *Інструкція*\n\n"
        "❗️ Наша повага до конфіденційності користувачів гарантує, що фотографії та історія запитів ніде не зберігаються."
    )
    await update.message.reply_text(welcome_message, parse_mode="Markdown")

# Обробник кнопки "Інструкція"
async def instruction(update: Update, context: ContextTypes.DEFAULT_TYPE):
    instruction_text = "📸 Щоб почати, просто надішли мені фото, і я підкажу, що робити далі!"
    await update.message.reply_text(instruction_text)

# Обробник кнопки "Про бота"
async def about(update: Update, context: ContextTypes.DEFAULT_TYPE):
    about_text = "ℹ️ Я бот, який допомагає взаємодіяти з різними інструментами та сервісами. Якщо є питання — пиши!"
    await update.message.reply_text(about_text)

# Обробник кнопки "Відправити Фото"
async def send_photo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Ви можете відправити фото, і я поверну його вам змонтованим!", reply_markup=ReplyKeyboardRemove())

# Функція для обробки та монтажу фото
async def handle_photo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    # Отримуємо фото з повідомлення
    photo = await update.message.photo[-1].get_file()  # Додано await для асинхронного завантаження
    photo_bytes = await photo.download_as_bytearray()  # Завантажуємо фото як байтовий масив

    # Відкриваємо зображення
    img = Image.open(BytesIO(photo_bytes))

    # Накладаємо чорно-білий фільтр
    img = img.convert("L")  # Конвертуємо зображення в чорно-біле

    # Зберігаємо оброблене зображення в пам'яті
    output = BytesIO()
    img.save(output, format="PNG")
    output.seek(0)

    # Відправляємо змонтоване фото назад
    await update.message.reply_photo(photo=output)

# Запуск бота
def main():
    app = ApplicationBuilder().token(TOKEN).build()

    # Обробники команд
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.Text(['👁 Інструкція']), instruction))
    app.add_handler(MessageHandler(filters.Text(['ℹ️ Про бота']), about))
    app.add_handler(MessageHandler(filters.Text(['📸 Відправити Фото']), send_photo))
    app.add_handler(MessageHandler(filters.PHOTO, handle_photo))  # Обробка фото

    app.run_polling()

if __name__ == '__main__':
    main()
