import asyncio
import logging
import sys
from aiogram import Bot, Dispatcher, F, Router
from aiogram.filters import CommandStart
from aiogram.types import CallbackQuery, InlineKeyboardButton, InlineKeyboardMarkup, Message

# === ТВОИ НАСТРОЙКИ (УЖЕ ВПИСАНЫ) ===
TOKEN = "9001003514:AAEW86X22hZRUykV7HjzeBJP2fPrlbX8jwA"
ADMIN_ID = 8441052897
CHANNEL_REVIEWS_URL = "https://t.me/твой_канал_отзывов" 
TIP_LINK = "https://www.tbank.ru/cf/..." 
SELLER_USERNAME = "w1nnnn31"
# =====================================

router = Router()

# Список товаров
PRODUCTS = {
    "so2": "🎮 Буст Standoff 2 (Калибровка/Ранг)",
    "bs": "👑 Буст Brawl Stars (Кубки/Ранг)",
    "metro": "☢️ Сопровождение PUBG Mobile (Metro Royale)",
    "acc": "🛒 Аккаунты и Голда"
}

BOOSTERS = {
    "W1nnn31": "👤 W1nnn31 (Топ игрок)",
    "Clan": "👥 Команда / Клан [TYPN]",
    "Random": "🎲 На усмотрение продавца"
}

def get_main_menu():
    return InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text="🛍 Каталог товаров и услуг", callback_data="catalog")],
            [InlineKeyboardButton(text="⭐ Отзывы клиентов", url=CHANNEL_REVIEWS_URL)],
            [InlineKeyboardButton(text="☕ Оставить чаевые", url=TIP_LINK)],
            [InlineKeyboardButton(text="📞 Связаться с продавцом", url=f"https://t.me/{SELLER_USERNAME}")]
        ]
    )

def get_catalog_menu():
    return InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text=PRODUCTS["so2"], callback_data="buy_so2")],
            [InlineKeyboardButton(text=PRODUCTS["bs"], callback_data="buy_bs")],
            [InlineKeyboardButton(text=PRODUCTS["metro"], callback_data="buy_metro")],
            [InlineKeyboardButton(text=PRODUCTS["acc"], callback_data="buy_acc")],
            [InlineKeyboardButton(text="⬅️ Назад в меню", callback_data="back_home")]
        ]
    )

def get_booster_menu(product_code: str):
    return InlineKeyboardMarkup(
        inline_keyboard=[
            [InlineKeyboardButton(text=BOOSTERS["W1nnn31"], callback_data=f"boost_{product_code}_W1nnn31")],
            [InlineKeyboardButton(text=BOOSTERS["Clan"], callback_data=f"boost_{product_code}_Clan")],
            [InlineKeyboardButton(text=BOOSTERS["Random"], callback_data=f"boost_{product_code}_Random")],
            [InlineKeyboardButton(text="⬅️ К списку товаров", callback_data="catalog")]
        ]
    )

@router.message(CommandStart())
async def cmd_start(message: Message):
    await message.answer(
        f"Привет, {message.from_user.first_name}! 👋\n\n"
        "Добро пожаловать в магазин игровых услуг **TyphOOn4k Store**.\n"
        "Выбери нужный раздел ниже:",
        reply_markup=get_main_menu(),
        parse_mode="Markdown"
    )

@router.callback_query(F.data == "back_home")
async def back_home(callback: CallbackQuery):
    await callback.message.edit_text(
        "Главное меню магазина:",
        reply_markup=get_main_menu(),
        parse_mode="Markdown"
    )
    await callback.answer()

@router.callback_query(F.data == "catalog")
async def show_catalog(callback: CallbackQuery):
    await callback.message.edit_text(
        "📦 **Каталог доступных услуг:**\nВыбери нужную игру или товар:",
        reply_markup=get_catalog_menu(),
        parse_mode="Markdown"
    )
    await callback.answer()

@router.callback_query(F.data.startswith("buy_"))
async def select_product(callback: CallbackQuery):
    product_code = callback.data.split("_")[1]
    product_title = PRODUCTS.get(product_code, "Товар")
    await callback.message.edit_text(
        f"Ты выбрал: **{product_title}**\n\nТеперь выбери исполнителя:",
        reply_markup=get_booster_menu(product_code),
        parse_mode="Markdown"
    )
    await callback.answer()

@router.callback_query(F.data.startswith("boost_"))
async def process_order(callback: CallbackQuery, bot: Bot):
    parts = callback.data.split("_")
    product_code = parts[1]
    booster_code = parts[2]

    product_title = PRODUCTS.get(product_code, product_code)
    booster_title = BOOSTERS.get(booster_code, booster_code)

    user = callback.from_user
    username_text = f"@{user.username}" if user.username else f"ID: {user.id}"

    # Ответ клиенту
    await callback.message.edit_text(
        "✅ **Заказ успешно оформлен!**\n\n"
        f"• **Услуга:** {product_title}\n"
        f"• **Исполнитель:** {booster_title}\n\n"
        f"Продавец скоро свяжется с тобой. Или можешь написать сам: @{SELLER_USERNAME}",
        reply_markup=get_main_menu(),
        parse_mode="Markdown"
    )

    # Уведомление тебе в личку
    admin_text = (
        "🚨 **НОВЫЙ ЗАКАЗ В МАГАЗИНЕ!**\n\n"
        f"👤 **Покупатель:** {user.full_name} ({username_text})\n"
        f"📦 **Товар:** {product_title}\n"
        f"⚡ **Бустер:** {booster_title}"
    )
    try:
        await bot.send_message(ADMIN_ID, admin_text, parse_mode="Markdown")
    except Exception as e:
        logging.error(f"Не удалось отправить уведомление админу: {e}")

    await callback.answer("Заказ отправлен продавцу!")

async def main():
    bot = Bot(token=TOKEN)
    dp = Dispatcher()
    dp.include_router(router)
    print("Бот успешно запущен!")
    await dp.start_polling(bot)

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO, stream=sys.stdout)
    asyncio.run(main())
