import datetime
import random
import hashlib
import sqlite3

# Database connection
db_connection = sqlite3.connect("bank_system.db")
db_cursor = db_connection.cursor()

# Create tables if they don't exist
db_cursor.execute("""
    CREATE TABLE IF NOT EXISTS accounts (
        account_number TEXT PRIMARY KEY,
        customer_name TEXT,
        balance REAL,
        is_locked INTEGER,
        password_hash TEXT
    )
""")
db_cursor.execute("""
    CREATE TABLE IF NOT EXISTS transactions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        account_number TEXT,
        description TEXT,
        amount REAL,
        timestamp TEXT
    )
""")

class BankSystem:
    def __init__(self):
        self.accounts = []
    
    def create_account(self, customer_name, initial_balance, password):
        account_number = self.generate_account_number()
        password_hash = self.hash_password(password)
        account = BankAccount(account_number, customer_name, initial_balance, password_hash)
        
        # Save account to the database
        db_cursor.execute("""
            INSERT INTO accounts (account_number, customer_name, balance, is_locked, password_hash)
            VALUES (?, ?, ?, ?, ?)
        """, (account_number, customer_name, initial_balance, 0, password_hash))
        db_connection.commit()
        
        return account
    
    def generate_account_number(self):
        # Generate a random account number
        account_number = "".join(str(random.randint(0, 9)) for _ in range(6))
        
        # Check if the account number already exists
        while self.find_account(account_number):
            account_number = "".join(str(random.randint(0, 9)) for _ in range(6))
        
        return account_number
    
    def find_account(self, account_number):
        # Find account in the database
        db_cursor.execute("""
            SELECT account_number, customer_name, balance, is_locked, password_hash
            FROM accounts
            WHERE account_number = ?
        """, (account_number,))
        account_data = db_cursor.fetchone()
        
        if account_data:
            return BankAccount(*account_data)
        else:
            return None
    
    def authenticate(self, account_number, password):
        account = self.find_account(account_number)
        if account:
            password_hash = self.hash_password(password)
            return account.password_hash == password_hash
        return False
    
    @staticmethod
    def hash_password(password):
        # Hash the password using a secure hashing algorithm (e.g., SHA-256)
        return hashlib.sha256(password.encode()).hexdigest()
    
    def deposit(self, account_number, amount):
        account = self.find_account(account_number)
        if account and account.is_unlocked():
            account.deposit(amount)
            
            # Update account balance in the database
            db_cursor.execute("""
                UPDATE accounts
                SET balance = ?
                WHERE account_number = ?
            """, (account.balance, account_number))
            db_connection.commit()
            
            # Record the transaction in the database
            self.record_transaction(account_number, "Deposit", amount)
            
            return True
        return False
    
    def withdraw(self, account_number, amount):
        account = self.find_account(account_number)
        if account and account.is_unlocked() and account.balance >= amount:
            account.withdraw(amount)
            
            # Update account balance in the database

