import tkinter as tk
from tkinter import messagebox, ttk
import matplotlib.pyplot as plt
import yfinance as yf
from datetime import datetime, timedelta

def get_exchange_rates(base_currency, target_currency, start_date, end_date):
    ticker_symbol = f"{base_currency}{target_currency}=X"
    data = yf.download(ticker_symbol, start=start_date, end=end_date)
    df = data[["Close"]].rename(columns={"Close": "Exchange Rate"})
    df.index.name = "Date"
    return df

def plot_exchange_rate(df):
    plt.figure(figsize=(10, 6))
    plt.plot(df.index, df['Exchange Rate'], marker='o', linestyle='-')
    plt.title('Exchange Rate Over Time')
    plt.xlabel('Date')
    plt.ylabel('Exchange Rate')
    plt.grid(True)
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

def get_currency_symbol(currency_code):
    currency_symbols = {
        "USD": "$",
        "EUR": "€",
        "GBP": "£",
        "JPY": "¥",
        "AUD": "A$",
        "CAD": "C$",
        "INR": "₹"
    }
    return currency_symbols.get(currency_code, currency_code)

def perform_analysis():
    try:
        base_currency = entry_from_currency.get().upper()
        target_currency = entry_to_currency.get().upper()
        start_date = datetime.strptime(entry_start_date.get(), "%Y-%m-%d")
        end_date = datetime.strptime(entry_end_date.get(), "%Y-%m-%d")

        start_date -= timedelta(days=365)  # Adjust start date to one year prior
        df = get_exchange_rates(base_currency, target_currency, start_date, end_date)
        plot_exchange_rate(df)
    except ValueError:
        messagebox.showerror("Error", "Please enter valid start and end dates in the format YYYY-MM-DD.")
    except Exception as e:
        messagebox.showerror("Error", str(e))

def convert():
    try:
        amount = float(entry_amount.get())
        from_currency = entry_from_currency.get().upper()
        to_currency = entry_to_currency.get().upper()
        ticker_symbol = f"{from_currency}{to_currency}=X"
        data = yf.download(ticker_symbol, period="1d")
        conversion_rate = data["Close"][-1]
        converted_amount = amount * conversion_rate
        decimal_precision = int(decimal_precision_var.get())

        from_currency_symbol = get_currency_symbol(from_currency)
        to_currency_symbol = get_currency_symbol(to_currency)

        label_result.config(text=f"{amount} {from_currency_symbol} = {converted_amount:.{decimal_precision}f} {to_currency_symbol}")
    except ValueError:
        label_result.config(text="Please enter a valid amount.")
    except Exception as e:
        messagebox.showerror("Error", str(e))

def get_todays_rate():
    try:
        from_currency = entry_from_currency.get().upper()
        to_currency = entry_to_currency.get().upper()
        ticker_symbol = f"{from_currency}{to_currency}=X"
        data = yf.download(ticker_symbol, period="1d")
        todays_rate = data["Close"][-1]
        from_currency_symbol = get_currency_symbol(from_currency)
        to_currency_symbol = get_currency_symbol(to_currency)
        label_result.config(text=f"Today's Rate ({from_currency_symbol} to {to_currency_symbol}): {todays_rate:.2f}")
    except Exception as e:
        messagebox.showerror("Error", str(e))

root = tk.Tk()
root.title("Currency Converter & Exchange Rate Analysis")
root.geometry("600x400")
root.configure(bg='#FFFFFF')

label_amount = tk.Label(root, text="Amount:", bg='#FFFFFF')
label_amount.grid(row=0, column=0, padx=10, pady=5)

entry_amount = tk.Entry(root)
entry_amount.grid(row=0, column=1, padx=10, pady=5)

label_from_currency = tk.Label(root, text="From Currency:", bg='#FFFFFF')
label_from_currency.grid(row=1, column=0, padx=10, pady=5)

entry_from_currency = ttk.Combobox(root, values=["USD", "EUR", "GBP", "JPY", "AUD", "CAD", "INR"])
entry_from_currency.grid(row=1, column=1, padx=10, pady=5)
entry_from_currency.current(0)

label_to_currency = tk.Label(root, text="To Currency:", bg='#FFFFFF')
label_to_currency.grid(row=2, column=0, padx=10, pady=5)

entry_to_currency = ttk.Combobox(root, values=["USD", "EUR", "GBP", "JPY", "AUD", "CAD", "INR"])
entry_to_currency.grid(row=2, column=1, padx=10, pady=5)
entry_to_currency.current(1)

button_convert = tk.Button(root, text="Convert", command=convert, bg="#003366", fg="white")
button_convert.grid(row=3, column=0, padx=10, pady=5)

button_clear = tk.Button(root, text="Clear", command=lambda: entry_amount.delete(0, tk.END), bg="#003366", fg="white")
button_clear.grid(row=3, column=1, padx=10, pady=5)

label_result = tk.Label(root, text="", bg='#FFFFFF', fg='#008000', font=('Helvetica', 12))
label_result.grid(row=4, column=0, columnspan=2, padx=10, pady=5)

label_start_date = tk.Label(root, text="Start Date (YYYY-MM-DD):", bg='#FFFFFF')
label_start_date.grid(row=5, column=0, padx=10, pady=5)

entry_start_date = tk.Entry(root)
entry_start_date.grid(row=5, column=1, padx=10, pady=5)

label_end_date = tk.Label(root, text="End Date (YYYY-MM-DD):", bg='#FFFFFF')
label_end_date.grid(row=6, column=0, padx=10, pady=5)

entry_end_date = tk.Entry(root)
entry_end_date.grid(row=6, column=1, padx=10, pady=5)

button_analyze = tk.Button(root, text="Analyze", command=perform_analysis, bg="#003366", fg="white")
button_analyze.grid(row=7, column=0, padx=10, pady=5)

button_todays_rate = tk.Button(root, text="Today's Rate", command=get_todays_rate, bg="#003366", fg="white")
button_todays_rate.grid(row=7, column=1, padx=10, pady=5)

label_info = tk.Label(root, text="Note: Dates should be in YYYY-MM-DD format.", bg='#FFFFFF', fg='#000080')
label_info.grid(row=8, column=0, columnspan=2, padx=10, pady=5)

# Add Theme Selection
theme_var = tk.StringVar()
theme_var.set("Light")

theme_label = tk.Label(root, text="Select Theme:", bg='#FFFFFF')
theme_label.grid(row=9, column=0, padx=10, pady=5)

theme_dropdown = ttk.Combobox(root, textvariable=theme_var, values=["Light", "Dark"], state="readonly")
theme_dropdown.grid(row=9, column=1, padx=10, pady=5)

def change_theme(event=None):
    selected_theme = theme_var.get()
    if selected_theme == "Light":
        root.configure(bg='#FFFFFF')
    elif selected_theme == "Dark":
        root.configure(bg='#000000')

theme_dropdown.bind("<<ComboboxSelected>>", change_theme)

# Add decimal precision dropdown
decimal_precision_var = tk.StringVar()
decimal_precision_var.set("2")

decimal_precision_label = tk.Label(root, text="Decimal Precision:", bg='#FFFFFF')
decimal_precision_label.grid(row=10, column=0, padx=10, pady=5)

decimal_precision_dropdown = ttk.Combobox(root, textvariable=decimal_precision_var, values=["0", "1", "2", "3", "4", "5"])
decimal_precision_dropdown.grid(row=10, column=1, padx=10, pady=5)

# Add date format dropdown
date_format_var = tk.StringVar()
date_format_var.set("YYYY-MM-DD")

date_format_label = tk.Label(root, text="Date Format:", bg='#FFFFFF')
date_format_label.grid(row=11, column=0, padx=10, pady=5)

date_format_dropdown = ttk.Combobox(root, textvariable=date_format_var, values=["YYYY-MM-DD", "MM-DD-YYYY", "DD-MM-YYYY"], state="readonly")
date_format_dropdown.grid(row=11, column=1, padx=10, pady=5)

def validate_date_format(date_text):
    if date_text == "" or date_text.count("-") != 2:
        return True
    try:
        datetime.strptime(date_text, "%Y-%m-%d")
        return True
    except ValueError:
        return False

def validate_date_format_dd_mm_yyyy(date_text):
    if date_text == "" or date_text.count("-") != 2:
        return True
    try:
        datetime.strptime(date_text, "%d-%m-%Y")
        return True
    except ValueError:
        return False

root.mainloop()
