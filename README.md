Personal Finance Transaction Tracker Project code :


CODE:

import pandas as pd
import csv
from datetime import datetime
from data_entry import get_date, get_amount, get_category, get_description
import matplotlib.pyplot as plt


#use Method - class to create a class named CSV and assign the name of the csv file to a variable named CSV_FILE. 
# This variable will be used to read the csv file in the future.
class CSV:
    CSV_FILE = "Financial_Data.csv"
    columns = ["date", "amount", "category","description"]
    FORMAT = "%d-%m-%Y"

    @classmethod # this would have access to the class but won't have access to its instance.
    def initialize_csv(cls):
        try:
            pd.read_csv(cls.CSV_FILE)
        except FileNotFoundError:
            df = pd.DataFrame(columns=cls.columns)
            df.to_csv(cls.CSV_FILE, index=False)

    @classmethod
    def add_entry(cls, date , amount, category, description):
        new_entry = {
            "date": date,
            "amount": amount,
            "category": category, 
            "description": description
        }
        with open(cls.CSV_FILE, "a", newline="") as csvfile: 
            #"a" stands for append mode, which allows you to add new data to the end of the file without overwriting existing data.
            writer  =  csv.DictWriter(csvfile, fieldnames=cls.columns) 
            # instead of repeating the column names, we can use the class variable "columns" to specify the fieldnames for the DictWriter.
            writer.writerow(new_entry)
        print("Entry added successfully!")

    @classmethod
    def get_transactions(cls, start_date, end_date):
        df = pd.read_csv(cls.CSV_FILE)
        df["date"] = pd.to_datetime(df["date"], format= CSV.FORMAT)
        start_date = datetime.strptime(start_date, CSV.FORMAT)
        end_date = datetime.strptime(end_date, CSV.FORMAT)
        
        mask = (df["date"] >= start_date) & (df["date"] <= end_date)
        filtered_df = df.loc[mask] #locates all the places where mask is added and returns the filtered dataframe.

        if filtered_df.empty:
            print("No transactions found in the specified date range.")
        else:
            print(f"Transactions from {start_date.strftime(CSV.FORMAT)} to {end_date.strftime(CSV.FORMAT)}:")
            print(
                filtered_df.to_string(
                    index=False , formatters = {"date": lambda x: x.strftime(CSV.FORMAT)}
                )
            )  # Display the filtered DataFrame without the index column
            
            total_income = filtered_df[filtered_df["category"] == "Income"]["amount"].sum()
            total_expense = filtered_df[filtered_df["category"] == "Expense"]["amount"].sum()
            print("\nSummary:") # for summarizing the total income and total expense in the specified date range.
            print(f"Total Income: ${total_income:.2f}") #:.2f is used to format the total income to 2 decimal places.
            print(f"Total Expense: ${total_expense:.2f}") #:.2f is used to format the total expense to 2 decimal places.
            print(f"Net Savings: ${total_income - total_expense:.2f}") # for calculating the net savings by subtracting total expense from total income and formatting it to 2 decimal places.

        return filtered_df  # Return the filtered DataFrame for further analysis if needed
def add():
    CSV.initialize_csv()
    date = get_date("Enter the date of the transaction (dd-mm-yyyy) or enter for today's date: ", allow_default=True)
    amount = get_amount()
    category = get_category()
    description = get_description()
    CSV.add_entry(date, amount, category, description)

CSV.get_transactions("01-01-2023", "31-12-2024")  # Example usage: Get transactions between 1st Jan 2024 and 31st Dec 2024

def plot_transactions(df):
    df.set_index('date', inplace=True)

    # Build a complete daily date range covering the full period (no gaps)
    daily_index = pd.date_range(df.index.min(), df.index.max(), freq='D')

    # resample groups transactions into daily buckets and sums the amount.
    # reindex fills in missing days with 0 so the chart has no gaps.
    income_df = (
        df[df["category"] == "Income"]
        .resample('D')
        .sum(numeric_only=True)
        .reindex(daily_index, fill_value=0)
    )

    expense_df = (
        df[df["category"] == "Expense"]
        .resample('D')
        .sum(numeric_only=True)
        .reindex(daily_index, fill_value=0)
    )

    # create a plot based on the income and expense df
    plt.figure(figsize=(9, 5))
    plt.plot(income_df.index, income_df["amount"], label="Income", color="green")
    plt.plot(expense_df.index, expense_df["amount"], label="Expense", color="red") 
    plt.xlabel("Date")
    plt.ylabel("Amount ($)")
    plt.title("Daily Income and Expenses")
    plt.legend()
    plt.grid(True)
    plt.show()


def main(): # main function to run the program and provide a menu for the user to choose between adding a new transaction or viewing transactions within a date range.
    while True:
        print("\n1. Add a new transaction")
        print("2. View transactions and a summary within a date range")
        print("3. Exit")
        choice = input("Enter your choice (1-3): ")

        if choice == "1":
            add()
        elif choice == "2":
            start_date = get_date("Enter the start date (dd-mm-yyyy): ")
            end_date = get_date("Enter the end date (dd-mm-yyyy): ")
            df = CSV.get_transactions(start_date, end_date) #defined it as a df to call it later for analysis and visualization.
            if input("Do you want to plot the transactions? (y/n): ").lower() == "y":
                plot_transactions(df) # calling the plot_transactions function to visualize the transactions if the user chooses to do so.
        elif choice == "3":
            print("Exiting the program.")
            break
        else:
            print("Invalid choice. Please enter a number between 1 and 3.")

if __name__ == "__main__":
    main()





output:


1. Add a new transaction
2. View transactions and a summary within a date range
3. Exit
Enter your choice (1-3): 2
Enter the start date (dd-mm-yyyy): 01-07-2024
Enter the end date (dd-mm-yyyy): 15-07-2024
Transactions from 01-07-2024 to 15-07-2024:
      date  amount category     description
01-07-2024  1000.0   Income          Salary
02-07-2024    50.0  Expense       Groceries
03-07-2024   150.0  Expense       Utilities
04-07-2024   200.0   Income  Freelance Work
05-07-2024    75.0  Expense      Restaurant
06-07-2024   100.0  Expense       Transport
07-07-2024    30.0  Expense          Snacks
08-07-2024   500.0   Income           Bonus
09-07-2024    90.0  Expense   Entertainment
10-07-2024    60.0  Expense          Coffee
11-07-2024   250.0   Income Stock Dividends
12-07-2024   120.0  Expense             Gas
13-07-2024    80.0  Expense           Books
14-07-2024   150.0  Expense        Clothing
15-07-2024  1100.0   Income          Salary

Summary:
Total Income: $3050.00
Total Expense: $905.00
Net Savings: $2145.00
Do you want to plot the transactions? (y/n): y
