import logging
import random
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import Application, CallbackQueryHandler, CommandHandler, ContextTypes

# 1. إعدادات التوكن والقنوات 🛠️
TOKEN = "8775989237:AAFrZHZKWyyswNnaGFXFvaMiRwGNZvW5l2M"

CHANNELS = {
    'ar': 'https://t.me/l_mahmoud_khaled_oraby_l',
    'en': '@اسم_قناتك_الانجليزية',
    'fr': '@اسم_قناتك_الفرنسية',
    'de': '@اسم_قناتك_الالمانية'
}

# 2. دالة إرسال الرسالة التلقائية ⏰
async def send_periodic_message(context: ContextTypes.DEFAULT_TYPE):
    job = context.job
    chat_id = job.chat_id
    user_lang = context.user_data.get('lang', 'ar') # الافتراضي عربي لو مختارش
    channel_id = CHANNELS.get(user_lang)

    try:
        # البوت بيعمل "Forward" لآخر رسالة من القناة للمستخدم
        # ملاحظة: لازم البوت يكون Admin في القناة
        await context.bot.forward_message(
            chat_id=chat_id,
            from_chat_id=channel_id,
            message_id=-1 # -1 تعني آخر رسالة (في بعض النسخ قد تحتاج لتعديل لجلب رسالة عشوائية)
        )
    except Exception as e:
        print(f"خطأ في الإرسال: {e}")

# 3. دالة البداية والترحيب ✉️
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [InlineKeyboardButton("العربية 🇪🇬", callback_data='ar')],
        [InlineKeyboardButton("English 🇺🇸", callback_data='en')],
        [InlineKeyboardButton("Français 🇫🇷", callback_data='fr')],
        [InlineKeyboardButton("Deutsch 🇩🇪", callback_data='de')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    
    await update.message.reply_text(
        "مرحباً بك! اختر لغتك لتفعيل الإرسال التلقائي:\n"
        "Welcome! Choose your language to activate auto-posts:",
        reply_markup=reply_markup
    )

# 4. معالجة اختيار اللغة وتشغيل المؤقت ⚙️
async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    
    selected_lang = query.data
    context.user_data['lang'] = selected_lang
    chat_id = update.effective_chat.id

    # إزالة أي وظائف قديمة للمستخدم ده عشان ميتكررش الإرسال
    current_jobs = context.job_queue.get_jobs_by_name(str(chat_id))
    for job in current_jobs:
        job.schedule_removal()

    # تشغيل المؤقت: يبعت رسالة كل 3600 ثانية (ساعة واحدة)
    context.job_queue.run_repeating(
        send_periodic_message, 
        interval=3600, 
        first=10, 
        chat_id=chat_id, 
        name=str(chat_id)
    )
    
    messages = {
        'ar': "✅ تم التفعيل! هبعتلك رسالة من القناة العربية كل ساعة.",
        'en': "✅ Activated! I will send you a post from the English channel every hour.",
        # أكمل باقي اللغات هنا بنفس الطريقة...
    }
    
    await query.edit_message_text(text=messages.get(selected_lang, "Done!"))

# 5. تشغيل البوت 🚀
def main():
    # بناء التطبيق مع تفعيل الـ JobQueue
    application = Application.builder().token(TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(button_handler))

    print("البوت يعمل الآن والمؤقت جاهز...")
    application.run_polling()

if __name__ == '__main__':
    main()
