from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes
import requests

TOKEN = "8596126878:AAFqMEpiFT9rAdigW5CDMTv5UUNzBwb11MU"

user_data = {}

# --- APIs ---
def get_rate(from_currency, to_currency):
    url = f"https://api.exchangerate-api.com/v4/latest/{from_currency}"
    data = requests.get(url).json()
    return data["rates"][to_currency]

def get_crypto(coin):
    url = f"https://api.coingecko.com/api/v3/simple/price?ids={coin}&vs_currencies=egp,usd"
    return requests.get(url).json()[coin]

# --- Start ---
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [InlineKeyboardButton("💱 تحويل عملات", callback_data="convert")],
        [InlineKeyboardButton("🪙 كريبتو", callback_data="crypto")],
        [InlineKeyboardButton("ℹ️ مساعدة", callback_data="help")]
    ]

    await update.message.reply_text(
        "👋 اهلا بيك في البوت الاحترافي للعملات",
        reply_markup=InlineKeyboardMarkup(keyboard)
    )

# --- Buttons ---
async def buttons(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    if query.data == "convert":
        user_data[query.from_user.id] = "waiting_amount"
        await query.message.reply_text("💱 ابعت المبلغ + العملة كده:\n100 USD EGP")

    elif query.data == "crypto":
        keyboard = [
            [InlineKeyboardButton("Bitcoin", callback_data="btc")],
            [InlineKeyboardButton("Litecoin", callback_data="ltc")],
            [InlineKeyboardButton("TON", callback_data="ton")]
        ]
        await query.message.reply_text(
            "اختار العملة:",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )

    elif query.data == "help":
        await query.message.reply_text(
            "📌 استخدم الأزرار\n💱 للتحويل\n🪙 للكريبتو"
        )

    elif query.data == "btc":
        data = get_crypto("bitcoin")
        await query.message.reply_text(f"🪙 BTC\nEGP: {data['egp']}\nUSD: {data['usd']}")

    elif query.data == "ltc":
        data = get_crypto("litecoin")
        await query.message.reply_text(f"🪙 LTC\nEGP: {data['egp']}\nUSD: {data['usd']}")

    elif query.data == "ton":
        data = get_crypto("toncoin")
        await query.message.reply_text(f"🪙 TON\nEGP: {data['egp']}\nUSD: {data['usd']}")

# --- تحويل ---
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id

    if user_id in user_data:
        try:
            parts = update.message.text.split()
            amount = float(parts[0])
            from_c = parts[1].upper()
            to_c = parts[2].upper()

            rate = get_rate(from_c, to_c)
            result = amount * rate

            await update.message.reply_text(
                f"💰 {amount} {from_c} = {result:.2f} {to_c}"
            )

            del user_data[user_id]

        except:
            await update.message.reply_text("❌ اكتب كده:\n100 USD EGP")

# --- Run ---
app = ApplicationBuilder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(CallbackQueryHandler(buttons))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

print("🚀 البوت الاحترافي شغال")
app.run_polling()# -
- We don't ask for money : the bot only detects and converts currencies.  قيمة صرف العملات الرقمية مقابل الدولار وغيره
