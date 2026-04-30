# Max-Meshkov
import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.expenses = []
        self.load_data()

        # Интерфейс
        tk.Label(root, text="Сумма:").grid(row=0, column=0, padx=5, pady=5)
        self.amount_entry = tk.Entry(root)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5)

        tk.Label(root, text="Категория:").grid(row=1, column=0, padx=5, pady=5)
        self.category_var = tk.StringVar()
        self.category_combo = ttk.Combobox(root, textvariable=self.category_var,
                                           values=["Еда", "Транспорт", "Развлечения", "Жильё", "Другое"])
        self.category_combo.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(root, text="Дата (ГГГГ-ММ-ДД):").grid(row=2, column=0, padx=5, pady=5)
        self.date_entry = tk.Entry(root)
        self.date_entry.grid(row=2, column=1, padx=5, pady=5)

        tk.Button(root, text="Добавить расход", command=self.add_expense).grid(row=3, column=0, columnspan=2, pady=10)

        # Таблица
        columns = ("Сумма", "Категория", "Дата")
        self.tree = ttk.Treeview(root, columns=columns, show="headings")
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=120)
        self.tree.grid(row=4, column=0, columnspan=2, padx=5, pady=5)

        # Фильтрация
        tk.Label(root, text="Фильтр по категории:").grid(row=5, column=0, padx=5, pady=5)
        self.filter_category_var = tk.StringVar()
        self.filter_category_combo = ttk.Combobox(root, textvariable=self.filter_category_var,
                                                   values=["Все"] + ["Еда", "Транспорт", "Развлечения", "Жильё", "Другое"])
        self.filter_category_combo.set("Все")
        self.filter_category_combo.grid(row=5, column=1, padx=5, pady=5)

        tk.Label(root, text="Период с (ГГГГ-ММ-ДД):").grid(row=6, column=0, padx=5, pady=5)
        self.start_date_entry = tk.Entry(root)
        self.start_date_entry.grid(row=6, column=1, padx=5, pady=5)

        tk.Label(root, text="по (ГГГГ-ММ-ДД):").grid(row=7, column=0, padx=5, pady=5)
        self.end_date_entry = tk.Entry(root)
        self.end_date_entry.grid(row=7, column=1, padx=5, pady=5)

        tk.Button(root, text="Применить фильтры", command=self.apply_filters).grid(row=8, column=0, pady=10)
        tk.Button(root, text="Сбросить фильтры", command=self.reset_filters).grid(row=8, column=1, pady=10)

        # Подсчёт суммы
        self.total_label = tk.Label(root, text="Общая сумма: 0")
        self.total_label.grid(row=9, column=0, columnspan=2, pady=5)

    def validate_input(self, amount_str, date_str):
        try:
            amount = float(amount_str)
            if amount <= 0:
                raise ValueError("Сумма должна быть положительным числом")
        except ValueError:
            messagebox.showerror("Ошибка", "Некорректная сумма")
            return False

        try:
            datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Некорректный формат даты (используйте ГГГГ-ММ-ДД)")
            return False
        return True

    def add_expense(self):
        amount_str = self.amount_entry.get()
        category = self.category_var.get()
        date_str = self.date_entry.get()

        if not self.validate_input(amount_str, date_str):
            return

        expense = {
            "amount": float(amount_str),
            "category": category,
            "date": date_str
        }
        self.expenses.append(expense)
        self.update_table()
        self.save_data()
        self.clear_entries()

    def clear_entries(self):
        self.amount_entry.delete(0, tk.END)
        self.category_var.set("")
        self.date_entry.delete(0, tk.END)

    def update_table(self):
        for item in self.tree.get_children():
            self.tree.delete(item)
        total = 0
        for expense in self.expenses:
            self.tree.insert("", "end", values=(expense["amount"], expense["category"], expense["date"]))
            total += expense["amount"]
        self.total_label.config(text=f"Общая сумма: {total}")

    def apply_filters(self):
        filtered = self.expenses
        category_filter = self.filter_category_var.get()
        if category_filter != "Все":
            filtered = [e for e in filtered if e["category"] == category_filter]

        start_date_str = self.start_date_entry.get()
        end_date_str = self.end_date_entry.get()

        if start_date_str:
            try:
                start_date = datetime.strptime(start_date_str, "%Y-%m-%d")
                filtered = [e for e in filtered if datetime.strptime(e["date"], "%Y-%m-%d") >= start_date]
            except ValueError:
                messagebox.showerror("Ошибка", "Некорректная начальная дата")
                return
        if end_date_str:
            try:
                end_date = datetime.strptime(end_date_str, "%Y-%m-%d")
                filtered = [e for e in filtered if datetime.strptime(e["date"], "%Y-%m-%d") <= end_date]
            except ValueError:
                messagebox.showerror("Ошибка", "Некорректная конечная дата")
                return

        for item in self.tree.get_children():
            self.tree.delete(item)
        total = 0
        for expense in filtered:
            self.tree.insert("", "end", values=(expense["amount"], expense["category"], expense["date"]))
            total += expense["amount"]
        self.total_label.config(text=f"Общая сумма (отфильтрованная): {total}")

    def reset_filters(self):
        self.filter_category_var.set("Все")
        self.start_date_entry.delete(0, tk.END)
        self.end_date_entry.delete(0, tk.END)
        self.update_table()

    def save_data(self):
        with open("expenses.json", "w", encoding="utf-8") as f:
            json.dump(self.expenses, f, ensure_ascii=False, indent=4)

    def load_data(self):
        try:
            with open("expenses.json", "r", encoding="utf-8") as f:
                self.expenses = json.load(f)
        except FileNotFoundError:
            self.expenses = []

if __name__ == "__main__":
    root = tk.Tk()
    app = ExpenseTracker(root)
