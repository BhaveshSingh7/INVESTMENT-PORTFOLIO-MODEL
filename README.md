# INVESTMENT-PORTFOLIO-MODEL
import pandas as pd
from tabulate import tabulate
import os

# File to store portfolio data
PORTFOLIO_FILE = "portfolio.csv"

def load_portfolio():
    """Load portfolio from CSV or create an empty DataFrame."""
    if os.path.exists(PORTFOLIO_FILE):
        return pd.read_csv(PORTFOLIO_FILE)
    return pd.DataFrame(columns=["Symbol", "Shares", "PurchasePrice", "CurrentPrice"])

def save_portfolio(df):
    """Save portfolio to CSV."""
    df.to_csv(PORTFOLIO_FILE, index=False)

def add_stock(df):
    """Add a new stock to the portfolio."""
    symbol = input("Enter stock symbol (e.g., AAPL): ").strip().upper()
    try:
        shares = int(input("Enter number of shares: "))
        purchase_price = float(input("Enter purchase price per share: "))
        current_price = float(input("Enter current price per share: "))

        if shares <= 0 or purchase_price < 0 or current_price < 0:
            print("Shares must be positive and prices cannot be negative.")
            return df

        new_stock = pd.DataFrame({
            "Symbol": [symbol],
            "Shares": [shares],
            "PurchasePrice": [purchase_price],
            "CurrentPrice": [current_price]
        })
        df = pd.concat([df, new_stock], ignore_index=True)
        print(f"Added {symbol} to portfolio.")
        return df
    except ValueError:
        print("Invalid input. Please enter numeric values for shares and prices.")
        return df

def remove_stock(df):
    """Remove a stock from the portfolio."""
    symbol = input("Enter stock symbol to remove: ").strip().upper()
    if symbol in df["Symbol"].values:
        df = df[df["Symbol"] != symbol]
        print(f"Removed {symbol} from portfolio.")
    else:
        print(f"Stock {symbol} not found in portfolio.")
    return df

def display_portfolio(df):
    """Display portfolio with performance metrics."""
    if df.empty:
        print("Portfolio is empty.")
        return

    # Calculate metrics
    df["CurrentValue"] = df["Shares"] * df["CurrentPrice"]
    df["GainLoss"] = (df["CurrentPrice"] - df["PurchasePrice"]) * df["Shares"]
    total_value = df["CurrentValue"].sum()
    total_invested = (df["Shares"] * df["PurchasePrice"]).sum()
    total_gain_loss = df["GainLoss"].sum()
    gain_loss_percent = (total_gain_loss / total_invested * 100) if total_invested > 0 else 0

    # Format for display
    display_df = df[["Symbol", "Shares", "PurchasePrice", "CurrentPrice", "CurrentValue", "GainLoss"]]
    display_df = display_df.round(2)

    print("\nYour Portfolio:")
    print(tabulate(display_df, headers=["Symbol", "Shares", "Purchase Price", "Current Price", "Current Value", "Gain/Loss"], tablefmt="pretty", floatfmt=".2f"))
    print(f"\nPortfolio Summary:")
    print(f"Total Portfolio Value: ${total_value:.2f}")
    print(f"Total Gain/Loss: ${total_gain_loss:.2f} ({gain_loss_percent:.2f}%)")

def main():
    """Main function to run the portfolio tracker."""
    df = load_portfolio()

    while True:
        print("\nStock Portfolio Performance Tracker")
        print("1. Add Stock")
        print("2. Remove Stock")
        print("3. View Portfolio")
        print("4. Exit")

        choice = input("Enter choice (1-4): ").strip()

        if choice == "1":
            df = add_stock(df)
            save_portfolio(df)
        elif choice == "2":
            df = remove_stock(df)
            save_portfolio(df)
        elif choice == "3":
            display_portfolio(df)
        elif choice == "4":
            print("Exiting...")
            break
        else:
            print("Invalid choice. Please select 1-4.")

if __name__ == "__main__":
    main()
