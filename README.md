pip install customtkinter requests
import customtkinter as ctk
import requests
import json
import os
from datetime import datetime

# Настройки окна
ctk.set_appearance_mode("dark")
ctk.set_default_color_theme("blue")

class CurrencyConverter(ctk.CTk):
    def __init__(self):
        super().__init__()
        self.title("Currency Converter")
        self.geometry("500x600")
        
        # Данные
        self.api_key = "ВАШ_КЛЮЧ_ЗДЕСЬ" # Замените на свой ключ
        self.history_file = "history.json"
        self.currencies = ["USD", "EUR", "RUB", "GBP", "JPY", "CNY"]
        
        self.create_widgets()
        self.load_history()

    def create_widgets(self):
        # Заголовок
        self.label = ctk.CTkLabel(self, text="Конвертер валют", font=("Arial", 24, "bold"))
        self.label.pack(pady=20)

        # Выбор валют
        self.frame_curr = ctk.CTkFrame(self)
        self.frame_curr.pack(pady=10)

        self.from_curr = ctk.CTkComboBox(self.frame_curr, values=self.currencies)
        self.from_curr.set("USD")
        self.from_curr.grid(row=0, column=0, padx=10)

        self.to_curr = ctk.CTkComboBox(self.frame_curr, values=self.currencies)
        self.to_curr.set("RUB")
        self.to_curr.grid(row=0, column=1, padx=10)

        # Ввод суммы
        self.amount_entry = ctk.CTkEntry(self, placeholder_text="Введите сумму")
        self.amount_entry.pack(pady=15)

        # Кнопка
        self.convert_btn = ctk.CTkButton(self, text="Конвертировать", command=self.convert)
        self.convert_btn.pack(pady=10)

        # Результат
        self.result_label = ctk.CTkLabel(self, text="", font=("Arial", 18))
        self.result_label.pack(pady=10)

        # История
        self.history_text = ctk.CTkTextbox(self, width=400, height=200)
        self.history_text.pack(pady=20)
        self.history_text.configure(state="disabled")

    def convert(self):
        amount = self.amount_entry.get()
        
        # Валидация
        try:
            amount = float(amount)
            if amount <= 0: raise ValueError
        except ValueError:
            self.result_label.configure(text="Ошибка: введите полож. число", text_color="red")
            return

        from_c = self.from_curr.get()
        to_c = self.to_curr.get()

        # Запрос к API
        url = f"https://exchangerate-api.com{self.api_key}/pair/{from_c}/{to_c}/{amount}"
        
        try:
            response = requests.get(url).json()
            if response['result'] == 'success':
                res = response['conversion_result']
                self.result_label.configure(text=f"{amount} {from_c} = {res:.2f} {to_c}", text_color="white")
                self.save_history(f"{datetime.now().strftime('%H:%M:%S')} - {amount} {from_c} -> {res:.2f} {to_c}")
            else:
                self.result_label.configure(text="Ошибка API", text_color="red")
        except Exception:
            self.result_label.configure(text="Нет связи с сервером", text_color="red")

    def save_history(self, record):
        history = []
        if os.path.exists(self.history_file):
            with open(self.history_file, 'r', encoding='utf-8') as f:
                history = json.load(f)
        
        history.append(record)
        with open(self.history_file, 'w', encoding='utf-8') as f:
            json.dump(history, f, ensure_ascii=False, indent=4)
        
        self.update_history_display(record)

    def load_history(self):
        if os.path.exists(self.history_file):
            with open(self.history_file, 'r', encoding='utf-8') as f:
                history = json.load(f)
                for item in history:
                    self.update_history_display(item)

    def update_history_display(self, record):
        self.history_text.configure(state="normal")
        self.history_text.insert("0.0", record + "\n")
        self.history_text.configure(state="disabled")

if __name__ == "__main__":
    app = CurrencyConverter()
    app.mainloop()
    # Currency Converter App

**Автор:** Ясмина Абаева

## Описание
Приложение для конвертации валют в реальном времени с сохранением истории запросов.

## Как получить API-ключ
1. Перейдите на сайт [exchangerate-api.com](https://exchangerate-api.com).
2. Введите свой email и нажмите "Get Free Key".
3. Подтвердите почту и скопируйте ваш персональный API Key.
4. Вставьте ключ в переменную `self.api_key` в коде программы.

## Запуск
```bash
python main.py
```
