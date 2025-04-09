[expenses.json](https://github.com/user-attachments/files/19667150/expenses.json)
[README.md](https://github.com/user-attachments/files/19667159/README.md)
 # Enhanced Personal Expense Tracker for College Students
# Includes budgeting, monthly reports, and category breakdown

import json
import os
import argparse
from datetime import datetime
from colorama import Fore, Style, init

# Initialize colorama for colored console output
init(autoreset=True)

# Constants
EXPENSES_FILE = "expenses.json"

class ExpenseTracker:
    def __init__(self):
        self.data = {
            "income": [],
            "fixed_expenses": [],
            "variable_expenses": [],
            "budget": 0
        }
        self.load_data()

    def load_data(self):
        if os.path.exists(EXPENSES_FILE):
            with open(EXPENSES_FILE, 'r') as file:
                self.data = json.load(file)

    def save_data(self):
        with open(EXPENSES_FILE, 'w') as file:
            json.dump(self.data, file, indent=4)

    def add_income(self, source, amount):
        self.data["income"].append({
            "source": source,
            "amount": amount,
            "date": datetime.now().strftime("%Y-%m-%d")
        })
        self.save_data()
        print(Fore.GREEN + "Income recorded successfully.")

    def add_expense(self, category_type, category, amount):
        if category_type not in ["fixed_expenses", "variable_expenses"]:
            print(Fore.RED + "Invalid category type.")
            return
        self.data[category_type].append({
            "category": category,
            "amount": amount,
            "date": datetime.now().strftime("%Y-%m-%d")
        })
        self.save_data()
        print(Fore.GREEN + "Expense recorded successfully.")
        self.check_budget_status()

    def set_budget(self, amount):
        self.data["budget"] = amount
        self.save_data()
        print(Fore.YELLOW + f"Monthly budget set to ₹{amount:.2f}")

    def check_budget_status(self):
        total_expense = sum(item['amount'] for item in self.data['fixed_expenses']) + \
                        sum(item['amount'] for item in self.data['variable_expenses'])
        if self.data["budget"] and total_expense > self.data["budget"]:
            print(Fore.RED + "⚠️ Budget exceeded!")

    def view_summary(self):
        total_income = sum(item['amount'] for item in self.data['income'])
        total_fixed = sum(item['amount'] for item in self.data['fixed_expenses'])
        total_variable = sum(item['amount'] for item in self.data['variable_expenses'])
        balance = total_income - (total_fixed + total_variable)

        print(Fore.CYAN + "\n========= EXPENSE SUMMARY =========")
        print(f"Total Income         : ₹{total_income:.2f}")
        print(f"Fixed Expenses       : ₹{total_fixed:.2f}")
        print(f"Variable Expenses    : ₹{total_variable:.2f}")
        print(f"Monthly Budget       : ₹{self.data['budget']:.2f}")
        print(Fore.YELLOW + f"Remaining Balance    : ₹{balance:.2f}")
        print(Fore.CYAN + "===================================")

    def report_by_month(self, month, year):
        def filter_by_date(entries):
            return [e for e in entries if datetime.strptime(e['date'], '%Y-%m-%d').month == month and datetime.strptime(e['date'], '%Y-%m-%d').year == year]

        fixed = filter_by_date(self.data['fixed_expenses'])
        variable = filter_by_date(self.data['variable_expenses'])
        income = filter_by_date(self.data['income'])

        print(Fore.CYAN + f"\n--- Report for {month:02}/{year} ---")
        for section, entries in zip(["Income", "Fixed", "Variable"], [income, fixed, variable]):
            print(Fore.MAGENTA + f"\n{section}:")
            for item in entries:
                label = item.get("source") or item.get("category")
                print(f"{item['date']} - {label}: ₹{item['amount']:.2f}")

    def category_stats(self):
        from collections import defaultdict

        category_totals = defaultdict(float)
        all_expenses = self.data['fixed_expenses'] + self.data['variable_expenses']

        for item in all_expenses:
            category_totals[item['category']] += item['amount']

        total = sum(category_totals.values())

        print(Fore.CYAN + "\n--- Expense Category Breakdown ---")
        for cat, amt in category_totals.items():
            percent = (amt / total) * 100 if total else 0
            bar = '█' * int(percent // 2)
            print(f"{cat}: ₹{amt:.2f} ({percent:.1f}%) {bar}")

    def list_all(self):
        print(Fore.CYAN + "\n--- Income ---")
        for item in self.data["income"]:
            print(f"{item['date']} - {item['source']}: ₹{item['amount']:.2f}")

        print(Fore.CYAN + "\n--- Fixed Expenses ---")
        for item in self.data["fixed_expenses"]:
            print(f"{item['date']} - {item['category']}: ₹{item['amount']:.2f}")

        print(Fore.CYAN + "\n--- Variable Expenses ---")
        for item in self.data["variable_expenses"]:
            print(f"{item['date']} - {item['category']}: ₹{item['amount']:.2f}")


def main():
    parser = argparse.ArgumentParser(description="Enhanced Personal Expense Tracker")
    parser.add_argument("--action", choices=[
        "add_income", "add_fixed", "add_variable", "summary",
        "list", "set_budget", "report", "stats"
    ], required=True, help="Action to perform")
    parser.add_argument("--category", help="Category for expense")
    parser.add_argument("--source", help="Source for income")
    parser.add_argument("--amount", type=float, help="Amount of income or expense")
    parser.add_argument("--month", type=int, help="Month for report")
    parser.add_argument("--year", type=int, help="Year for report")

    args = parser.parse_args()
    tracker = ExpenseTracker()

    if args.action == "add_income" and args.source and args.amount is not None:
        tracker.add_income(args.source, args.amount)
    elif args.action == "add_fixed" and args.category and args.amount is not None:
        tracker.add_expense("fixed_expenses", args.category, args.amount)
    elif args.action == "add_variable" and args.category and args.amount is not None:
        tracker.add_expense("variable_expenses", args.category, args.amount)
    elif args.action == "set_budget" and args.amount is not None:
        tracker.set_budget(args.amount)
    elif args.action == "summary":
        tracker.view_summary()
    elif args.action == "list":
        tracker.list_all()
    elif args.action == "report" and args.month and args.year:
        tracker.report_by_month(args.month, args.year)
    elif args.action == "stats":
        tracker.category_stats()
    else:
        print(Fore.RED + "Invalid input. Use --help for guidance.")

if __name__ == "__main__":
    main()
