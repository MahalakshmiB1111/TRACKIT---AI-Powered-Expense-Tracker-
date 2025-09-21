# TRACKIT---AI-Powered-Expense-Tracker-



import os
import tkinter as tk
from tkinter import ttk
from tkinter import messagebox
from tkcalendar import Calendar
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from sklearn.linear_model import LinearRegression
import numpy as np

analysis_window = None

# Global variable for the budget
budget = None

# Function to add an expense
def add_expense():
    date = date_entry.get()
    category = category_entry.get()
    amount = amount_entry.get()

    if date and category and amount:
        try:
            amount_float = float(amount)  # Validate amount
            with open("expenses.txt", "a") as file:
                file.write(f"{date},{category},{amount_float}\n")
            status_label.config(text="Added successfully!", fg="green")
            date_entry.delete(0, tk.END)
            category_entry.delete(0, tk.END)
            amount_entry.delete(0, tk.END)
            view_expenses()
            check_budget()
        except ValueError:
            status_label.config(text="Amount must be a number!", fg="red")
    else:
        status_label.config(text="Please fill all the fields!", fg="red")

# Function to delete an expense
def delete_expense():
    selected_item = expenses_tree.selection()
    if selected_item:
        item_text = expenses_tree.item(selected_item, "values")
        date, category, amount = item_text
        with open("expenses.txt", "r") as file:
            lines = file.readlines()
        with open("expenses.txt", "w") as file:
            for line in lines:
                if line.strip() != f"{date},{category},{amount}":
                    file.write(line)
        status_label.config(text="Deleted successfully!", fg="green")
        view_expenses()
        check_budget()
    else:
        status_label.config(text="Select an expense to delete!", fg="red")

# Function to view expenses
def view_expenses():
    if os.path.exists("expenses.txt"):
        total_expense = 0
        expenses_tree.delete(*expenses_tree.get_children())
        with open("expenses.txt", "r") as file:
            for line in file:
                date, category, amount = line.strip().split(",")
                expenses_tree.insert("", tk.END, values=(date, category, amount))
                total_expense += float(amount)
        total_label.config(text=f"Total Expense: {total_expense:.2f}")
    else:
        total_label.config(text="No expenses recorded.")
        expenses_tree.delete(*expenses_tree.get_children())

from sklearn.linear_model import LinearRegression
import numpy as np

# Fuunction to predict future expenses
def predict_expenses():
    if os.path.exists("expenses.txt"):
        dates = []
        amounts = []

        with open("expenses.txt", "r") as file:
            for line in file:
                date, _, amount = line.strip().split(",")
                day = int(date.split("-")[0])  # Use day as a feature
                dates.append(day)
                amounts.append(float(amount))

        if len(dates) < 2:  # Ensure enough data for prediction
            status_label.config(text="Not enough data for prediction.", fg="red")
            return

        # Prepare data for the model
        X = np.array(dates).reshape(-1, 1)  # Days as input
        y = np.array(amounts)  # Amounts as output

        # Train a linear regression model
        model = LinearRegression()
        model.fit(X, y)

        # Predict expenses for the next 7 days
        future_days = np.array(range(max(dates) + 1, max(dates) + 8)).reshape(-1, 1)
        predictions = model.predict(future_days)

        # Update the status label with predicted expenses
        predicted_expenses = ", ".join([f"{pred:.2f}" for pred in predictions])
        status_label.config(
            text=f"Predicted Expenses for next 7 days: {predicted_expenses}", fg="blue"
        )

        # Visualization
        fig, ax = plt.subplots(figsize=(6, 4), dpi=100)
        ax.plot(dates, amounts, 'bo-', label="Historical Data")
        ax.plot(future_days.flatten(), predictions, 'r--', label="Predicted Expenses")
        ax.set_title("Expense Prediction")
        ax.set_xlabel("Days")
        ax.set_ylabel("Amount")
        ax.legend()

        # Add labels for predicted points
        for i, (day, pred) in enumerate(zip(future_days.flatten(), predictions)):
            ax.annotate(f"{pred:.2f}", (day, pred), textcoords="offset points", xytext=(0, 5), ha='center', fontsize=9, color="red")

        # Embed the plot in the Tkinter window
        prediction_window = tk.Toplevel(root)
        prediction_window.title("Expense Prediction")
        canvas = FigureCanvasTkAgg(fig, master=prediction_window)
        canvas_widget = canvas.get_tk_widget()
        canvas_widget.pack(pady=20)
        canvas.draw()

    else:
        status_label.config(text="No expenses recorded to predict.", fg="red")
def show_analysis():
    global analysis_window
    if analysis_window and tk.Toplevel.winfo_exists(analysis_window):
        analysis_window.lift()  # Bring the window to the front
        return

    analysis_window = tk.Toplevel(root)
    analysis_window.title("Expense Analysis")
    analysis_window.geometry("300x300")
    analysis_window.config(bg="yellow")

    refresh_analysis()  # Show the initial analysis

def refresh_analysis():
    global analysis_window
    if not analysis_window or not tk.Toplevel.winfo_exists(analysis_window):
        return  # If the analysis window is not open, do nothing

    for widget in analysis_window.winfo_children():
        widget.destroy()  # Clear the previous contents of the window

    if os.path.exists("expenses.txt"):
        categories = {}
        with open("expenses.txt", "r") as file:
            for line in file:
                _, category, amount = line.strip().split(",")
                amount = float(amount)
                categories[category] = categories.get(category, 0) + amount

        if categories:
            # Generate the pie chart
            fig, ax = plt.subplots(figsize=(6, 4), dpi=100)
            ax.pie(categories.values(), labels=categories.keys(), autopct="%1.1f%%", startangle=140)
            ax.set_title("Expenses by Category")

            # Embed the plot in the Tkinter window
            canvas = FigureCanvasTkAgg(fig, master=analysis_window)
            canvas_widget = canvas.get_tk_widget()
            canvas_widget.pack(pady=20)
            canvas.draw()
        else:
            tk.Label(analysis_window, text="No expenses recorded to visualize.", bg="lightblue", fg="red").pack(pady=20)
    else:
        tk.Label(analysis_window, text="No expenses recorded to visualize.", bg="lightblue", fg="red").pack(pady=20)

# Function to set the budget
def set_budget():
    global budget
    budget_input = budget_entry.get()

    try:
        budget = float(budget_input)
        with open("budget.txt", "w") as file:
            file.write(str(budget))  # Save the budget to a file
        status_label.config(text=f"Budget set to: {budget:.2f}", fg="green")
        budget_entry.delete(0, tk.END)
        check_budget()
    except ValueError:
        status_label.config(text="Please enter a valid budget amount!", fg="red")

# Function to load the budget from file on application start
def load_budget():
    global budget
    if os.path.exists("budget.txt"):
        with open("budget.txt", "r") as file:
            try:
                budget = float(file.read())
                status_label.config(text=f"Budget loaded: {budget:.2f}", fg="blue")
            except ValueError:
                status_label.config(text="Error loading budget. Please set a new one.", fg="red")
                budget = None
    else:
        status_label.config(text="No budget set. Please enter one.", fg="red")
        budget = None



# Update check_budget to use the loaded budget
def check_budget():
    global budget
    if budget is not None and os.path.exists("expenses.txt"):
        total_expense = 0
        with open("expenses.txt", "r") as file:
            for line in file:
                _, _, amount = line.strip().split(",")
                total_expense += float(amount)

        remaining_budget = budget - total_expense
        if remaining_budget < 0:
            status_label.config(text=f"Over budget by {abs(remaining_budget):.2f}", fg="red")
        else:
            status_label.config(text=f"Remaining Budget: {remaining_budget:.2f}", fg="blue")


# Function to show the calendar and select a date
def open_calendar():
    calendar_window = tk.Toplevel(root)
    calendar_window.title("Select a Date")
    calendar_window.geometry("300x300")
    calendar = Calendar(calendar_window, selectmode='day', date_pattern='dd-mm-yyyy')
    calendar.pack(pady=20)

    def select_date():
        selected_date = calendar.get_date()
        date_entry.delete(0, tk.END)
        date_entry.insert(0, selected_date)
        calendar_window.destroy()

    select_button = tk.Button(calendar_window, text="Select Date", command=select_date)
    select_button.pack(pady=10)

# Function to open chatbot
def open_chatbot():
    chatbot_window = tk.Toplevel(root)
    chatbot_window.title("Chatbot")
    chatbot_window.geometry("500x300")
    chatbot_window.config(bg="midnightblue")

    # Chatbot response display
    chatbot_response = tk.Text(chatbot_window, height=12, width=70, wrap="word", bg="lightyellow", fg="black")
    chatbot_response.pack(pady=10)
    chatbot_response.insert(tk.END, "Welcome to TrackIt! How can I assist you?\n")

    # Entry field for user input
    chatbot_entry = tk.Entry(chatbot_window, width=50, bg="white", fg="black")
    chatbot_entry.pack(pady=10)
    
    # Button to send input
    chatbot_button = tk.Button(chatbot_window, text="Send", command=lambda: process_chatbot_input())
    chatbot_button.pack(pady=10)

    # Handle chatbot input
    def process_chatbot_input(event=None):
        chatbot_input = chatbot_entry.get().strip()  # Get user input
        chatbot_entry.delete(0, tk.END)  # Clear entry field

        if chatbot_input:  # Ensure input is not empty
            chatbot_response.insert(tk.END, f"You: {chatbot_input}\n")  # Show user input in the response box
            response = generate_response(chatbot_input)  # Generate chatbot response
            chatbot_response.insert(tk.END, f"Bot: {response}\n\n")  # Show bot response

    # Define the chatbot response logic
    def generate_response(user_input):
        user_input = user_input.lower()  # Normalize input for consistent matching

        # Predefined responses for common commands
        responses = {
            "add": "To add an expense, please fill in the date, category, and amount fields in the main window and click 'Add Expense'.",
            "delete": "To delete an expense, select it from the list in the main window and click 'Delete Expense'.",
            "view": "You can view all recorded expenses in the list displayed in the main window.",
            "visualize": "Click on the 'Show Analysis' button in the main window to visualize your expenses by category.",
            "budget": "You can set a budget in the main window, and the application will notify you if you exceed it.",
            "predict": (
                "The 'Predict Analysis' feature uses past data to forecast your spending for the next 7 days. "
                "Click on 'Predict Spending' in the main window to see the prediction."
            ),
            "analysis": (
                "The 'Show Analysis' feature provides a detailed visualization of your expenses by category. "
                "You can access this through the main window."
            ),
            "hi": "Hello! How can I assist you today?",
            "hello": "Hi there! Feel free to ask me anything about using TrackIt.",
            "thank you": "You're welcome! I'm here to help anytime. If you're done, type 'exit' to close the chatbot.",
            "exit": "Exiting the chatbot. Have a great day!",
            "help": (
                "Here are some commands you can try:\n"
                "- 'Add': Learn how to add an expense.\n"
                "- 'Delete': Learn how to delete an expense.\n"
                "- 'View': Find out how to view expenses.\n"
                "- 'Visualize': Learn how to visualize expenses.\n"
                "- 'Budget': Learn how to set or manage your budget.\n"
                "- 'Predict': Learn about predicting future spending.\n"
                "- 'Analysis': Learn how to view detailed spending analysis."
            ),
        }

        # Match user input to a predefined response
        for keyword, response in responses.items():
            if keyword in user_input:
                if keyword == "exit":  # Special handling for exiting the chatbot
                    chatbot_window.destroy()
                    return "Chatbot session ended."
                return response

        # Fallback response for unrecognized input
        return "I'm sorry, I didn't quite understand that. Please type 'help' for a list of available commands."

    # Bind the <Return> key to process input
    chatbot_entry.bind('<Return>', process_chatbot_input)


# Main application window
root = tk.Tk()
root.title("Expense Tracker")
root.config(bg="turquoise")

header_label = tk.Label(root, text="Welcome to Expense Tracker", font=("Helvetica", 16, "bold"), bg="lightblue")
header_label.grid(row=0, column=1, columnspan=1, padx=10, pady=20)

# Add labels and entries for budget
budget_label = tk.Label(root, text="Add Budget:")
budget_label.grid(row=1, column=0, padx=6, pady=7)
budget_entry = tk.Entry(root)
budget_entry.grid(row=1, column=2, padx=7, pady=7)

budget_button = tk.Button(root, text="Set  Budget", command=set_budget)
budget_button.grid(row=2, column=1, padx=8, pady=10)

budget_status_label = tk.Label(root, text="", fg="green")
budget_status_label.grid(row=2, column=0, columnspan=3, padx=5, pady=5)

# Create labels and entries for adding expenses
date_label = tk.Label(root, text="Date (DD-MM-YYYY):")
date_label.grid(row=3, column=0, padx=6, pady=7)
date_entry = tk.Entry(root)
date_entry.grid(row=3, column=2, padx=7, pady=7)

category_label = tk.Label(root, text="Category:")
category_label.grid(row=4, column=0, padx=6, pady=7)
category_entry = tk.Entry(root)
category_entry.grid(row=4, column=2, padx=7, pady=7)

amount_label = tk.Label(root, text="Amount:")
amount_label.grid(row=5, column=0, padx=6, pady=7)
amount_entry = tk.Entry(root)
amount_entry.grid(row=5, column=2, padx=7, pady=7)

add_button = tk.Button(root, text="Add Expense", command=add_expense)
add_button.grid(row=6, column=0, columnspan=3, padx=8, pady=10)

# Create a treeview to display expenses
columns = ("Date", "Category", "Amount")
expenses_tree = ttk.Treeview(root, columns=columns, show="headings")
expenses_tree.heading("Date", text="Date")
expenses_tree.heading("Category", text="Category")
expenses_tree.heading("Amount", text="Amount")
expenses_tree.grid(row=7, column=0, columnspan=3, padx=8, pady=8)

# Create a label to display the total expense
total_label = tk.Label(root, text="")
total_label.grid(row=8, column=0, columnspan=3, padx=8, pady=8)

# Status label
status_label = tk.Label(root, text="", fg="yellow")
status_label.grid(row=9, column=0, columnspan=3, padx=5, pady=5)

# Add a button to open the calendar
calendar_button = tk.Button(root, text="📅", command=open_calendar)
calendar_button.grid(row=3, column=3, padx=5, pady=7)

# Buttons for viewing, deleting, visualizing expenses, and opening chatbot


delete_button = tk.Button(root, text="Delete Expense", command=delete_expense)
delete_button.grid(row=11, column=2, padx=5, pady=10)

# Check if the 'expenses.txt' file exists; create it if it doesn't
if not os.path.exists("expenses.txt"):
    with open("expenses.txt", "w"):
        pass

# Display existing expenses on application start
view_expenses()
load_budget()
# Add a button to predict expenses
predict_button = tk.Button(root, text="Predict", command=predict_expenses)
predict_button.grid(row=10, column=2, padx=5, pady=10)

analysis_button = tk.Button(root, text="Analysis", command=show_analysis)
analysis_button.grid(row=10, column=0, columnspan=1, padx=5, pady=10)

chatbot_button = tk.Button(root, text="Open Chatbot", command=open_chatbot)
chatbot_button.grid(row=11, column=0, columnspan=1, padx=5, pady=10)

# Start the Tkinter main loop
root.mainloop()   
