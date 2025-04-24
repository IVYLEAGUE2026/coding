import asyncio
import logging
from aiogram import Bot, Dispatcher, types
from aiogram.enums import ParseMode
from aiogram.types import Message
from aiogram.filters import CommandStart, Command
from aiogram.client.bot import DefaultBotProperties
import random

API_TOKEN = "ТокенВашегоТелеграмБота"

logging.basicConfig(level=logging.INFO)

bot = Bot(token=API_TOKEN, default=DefaultBotProperties(parse_mode=ParseMode.HTML))
dp = Dispatcher()

SYMPTOM_DB = {
    "горло": [
        {"disease": "фарингит", "probability": 70, "treatment": "Полоскание горла солёной водой, при необходимости — антибиотики."},
        {"disease": "ангина", "probability": 50, "treatment": "Антибиотики, отдых и обильное питьё."},
        {"disease": "ОРВИ", "probability": 60, "treatment": "Жаропонижающие, отдых, много жидкости."},
        {"disease": "ларингит", "probability": 60, "treatment": "Отдых, покой для голосовых связок, ингаляции."},
        {"disease": "тонзиллит", "probability": 65, "treatment": "Антибиотики, полоскание, покой."}
    ],
    "голова": [
        {"disease": "мигрень", "probability": 80, "treatment": "Обезболивающие, темнота, покой."},
        {"disease": "стресс", "probability": 60, "treatment": "Расслабление, дыхательные упражнения, возможно успокоительные."},
        {"disease": "обезвоживание", "probability": 50, "treatment": "Регидратация (пить много воды)."},
        {"disease": "гипертония", "probability": 75, "treatment": "Контроль давления, лекарства от гипертонии."},
        {"disease": "сотрясение мозга", "probability": 50, "treatment": "Отдых, наблюдение, возможно госпитализация."}
    ],
    "температура": [
        {"disease": "грипп", "probability": 90, "treatment": "Жаропонижающие, отдых, много жидкости."},
        {"disease": "ОРВИ", "probability": 60, "treatment": "Полоскание горла, жаропонижающие."},
        {"disease": "токсиновые инфекции", "probability": 40, "treatment": "Обильное питьё, отдых, антибиотики при необходимости."},
        {"disease": "пневмония", "probability": 80, "treatment": "Антибиотики, обильное питьё, покой."},
        {"disease": "сепсис", "probability": 30, "treatment": "Неотложная медицинская помощь, антибиотики, госпитализация."}
    ],
    "недосып": [
        {"disease": "астения", "probability": 70, "treatment": "Нормализация режима сна, отдых, витаминизация."},
        {"disease": "перегорание", "probability": 60, "treatment": "Психотерапия, отдых, работа с режимом."},
        {"disease": "депрессия", "probability": 50, "treatment": "Психологическая помощь, антидепрессанты, нормализация режима."},
        {"disease": "синдром хронической усталости", "probability": 60, "treatment": "Психологическая помощь, нормализация режима сна."},
        {"disease": "нарушение циркадных ритмов", "probability": 55, "treatment": "Коррекция режима сна, приём мелатонина."}
    ],
    "рвота": [
        {"disease": "пищевое отравление", "probability": 80, "treatment": "Питьё регидратационных растворов, покой."},
        {"disease": "вирусная инфекция", "probability": 70, "treatment": "Обильное питьё, антисептики, антибиотики (если назначено врачом)."},
        {"disease": "гастрит", "probability": 60, "treatment": "Антибиотики, строгая диета."},
        {"disease": "язва желудка", "probability": 70, "treatment": "Диета, антисекреторные средства, возможно антибиотики."},
        {"disease": "панкреатит", "probability": 65, "treatment": "Нулевой режим (отказ от еды), восстановление через пару дней."}
    ],
    "сонливость": [
        {"disease": "недосыпание", "probability": 85, "treatment": "Регулярный сон, восстановление режима."},
        {"disease": "анемия", "probability": 50, "treatment": "Приём препаратов железа, коррекция питания."},
        {"disease": "синдром хронической усталости", "probability": 60, "treatment": "Психологическая помощь, нормализация режима сна."},
        {"disease": "гипотиреоз", "probability": 65, "treatment": "Лечение щитовидной железы, гормональная терапия."},
        {"disease": "депрессия", "probability": 50, "treatment": "Психологическая помощь, антидепрессанты."}
    ],
    "боль в мышцах": [
        {"disease": "миозит", "probability": 75, "treatment": "Противовоспалительные препараты, отдых, физиотерапия."},
        {"disease": "травма", "probability": 60, "treatment": "Покой, лед на место повреждения, обезболивающие."},
        {"disease": "фибромиалгия", "probability": 50, "treatment": "Антидепрессанты, физиотерапия, расслабляющие упражнения."}
    ],
    "спазмы": [
        {"disease": "расстройства пищеварения", "probability": 70, "treatment": "Диета, спазмолитики."},
        {"disease": "опухоль", "probability": 50, "treatment": "Диагностика, возможно оперативное вмешательство."},
        {"disease": "пищеводный рефлюкс", "probability": 60, "treatment": "Антациды, изменение питания."}
    ],
    "колющая боль в области груди": [
        {"disease": "инфаркт миокарда", "probability": 90, "treatment": "Немедленное обращение к врачу, госпитализация."},
        {"disease": "пневмония", "probability": 65, "treatment": "Антибиотики, обильное питьё, покой."},
        {"disease": "межреберная невралгия", "probability": 70, "treatment": "Противовоспалительные препараты, физиотерапия."}
    ],
}

@dp.message(CommandStart())
async def cmd_start(message: Message):
    await message.answer("👋 Привет! Напиши свои симптомы, а я помогу определить возможный диагноз.\n\n"
                         "Ты можешь попробовать следующие команды:\n"
                         "/symptoms - Список симптомов\n"
                         "/help - Как использовать бота")

@dp.message(Command("symptoms"))
async def cmd_symptoms(message: Message):
    symptoms = "Вот список симптомов, с которыми я могу помочь:\n"
    for symptom in SYMPTOM_DB.keys():
        symptoms += f"- {symptom}\n"
    await message.answer(symptoms)

@dp.message(Command("help"))
async def cmd_help(message: Message):
    await message.answer(
        "Я могу помочь вам с диагностикой заболеваний на основе симптомов.\n"
        "Просто напишите свои симптомы, и я предложу возможные диагнозы. Вот пример:\n"
        "'У меня болит горло и голова.'\n"
        "Также вы можете использовать команды:\n"
        "/symptoms - Список симптомов\n"
        "/help - Помощь по использованию бота"
    )

@dp.message()
async def diagnose(message: Message):
    text = message.text.lower()
    diagnoses = []
    treatments = []
    probabilities = []

    for keyword, possible_diseases in SYMPTOM_DB.items():
        if keyword in text:
            for disease in possible_diseases:
                diagnoses.append(disease["disease"])
                treatments.append(disease["treatment"])
                probabilities.append(disease["probability"])

    if diagnoses:
        response = "🤔 Возможно, это:\n"
        for i, disease in enumerate(set(diagnoses)):
            response += f"\n- {disease} (Вероятность: {probabilities[i]}%)"
            response += f"\n  Способ лечения: {treatments[i]}"

        response += "\n\nНо это не точный диагноз, и я рекомендую обратиться к врачу для точной диагностики!"
    else:
        response = "😕 Не смог ничего точно определить по этому описанию. Может быть, это не входит в мой список."

    await message.answer(response)

async def main():
    await bot.delete_webhook(drop_pending_updates=True)
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
