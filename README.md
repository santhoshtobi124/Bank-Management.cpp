#include <iostream>
#include <fstream>
#include <iomanip>
#include <string>
using namespace std;

class BankAccount {
private:
    int accountNumber;
    string name;
    double balance;

public:
    BankAccount() {
        accountNumber = 0;
        name = "";
        balance = 0.0;
    }

    BankAccount(int accNo, string n, double bal) {
        accountNumber = accNo;
        name = n;
        balance = bal;
    }

    void createAccount() {
        cout << "\nEnter Account Number: ";
        cin >> accountNumber;

        cin.ignore();
        cout << "Enter Customer Name: ";
        getline(cin, name);

        cout << "Enter Initial Deposit: ";
        cin >> balance;

        if (balance < 0) {
            cout << "Invalid amount!\n";
            balance = 0;
        }
    }

    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            cout << "Amount deposited successfully.\n";
        } else {
            cout << "Invalid deposit amount.\n";
        }
    }

    bool withdraw(double amount) {
        if (amount <= 0) {
            cout << "Invalid withdrawal amount.\n";
            return false;
        }

        if (amount > balance) {
            cout << "Insufficient balance.\n";
            return false;
        }

        balance -= amount;
        cout << "Amount withdrawn successfully.\n";
        return true;
    }

    void display() const {
        cout << "\n--------------------------------\n";
        cout << "Account Number : " << accountNumber << endl;
        cout << "Customer Name  : " << name << endl;
        cout << fixed << setprecision(2);
        cout << "Balance        : Rs. " << balance << endl;
        cout << "--------------------------------\n";
    }

    int getAccountNumber() const {
        return accountNumber;
    }

    // Save account data to file
    void saveToFile() const {
        ofstream file("accounts.dat", ios::app);

        if (file.is_open()) {
            file << accountNumber << "|"
                 << name << "|"
                 << balance << endl;
            file.close();
        }
    }
};

// Find account from file
bool findAccount(int accNo, BankAccount &account) {
    ifstream file("accounts.dat");

    if (!file.is_open())
        return false;

    int number;
    string name;
    double balance;

    while (file >> number) {
        file.ignore();
        getline(file, name, '|');
        file >> balance;
        file.ignore();

        if (number == accNo) {
            account = BankAccount(number, name, balance);
            file.close();
            return true;
        }
    }

    file.close();
    return false;
}

// Rewrite file after updating account
void updateFile(const BankAccount &updatedAccount) {
    ifstream input("accounts.dat");
    ofstream temp("temp.dat");

    int number;
    string name;
    double balance;

    while (input >> number) {
        input.ignore();
        getline(input, name, '|');
        input >> balance;
        input.ignore();

        if (number == updatedAccount.getAccountNumber()) {
            updatedAccount.display();

            // Save updated record
            // We need access to the original account details,
            // so create a temporary object.
            temp << updatedAccount.getAccountNumber()
                 << "|" << name << "|" << balance << endl;
        } else {
            temp << number << "|" << name << "|" << balance << endl;
        }
    }

    input.close();
    temp.close();

    remove("accounts.dat");
    rename("temp.dat", "accounts.dat");
}

void updateAccountBalance(int accNo, double amount, bool isDeposit) {
    BankAccount account;

    if (!findAccount(accNo, account)) {
        cout << "Account not found!\n";
        return;
    }

    if (isDeposit) {
        account.deposit(amount);
    } else {
        if (!account.withdraw(amount))
            return;
    }

    // Rewrite complete file correctly
    ifstream input("accounts.dat");
    ofstream temp("temp.dat");

    int number;
    string name;
    double balance;

    while (input >> number) {
        input.ignore();
        getline(input, name, '|');
        input >> balance;
        input.ignore();

        if (number == accNo) {
            temp << number << "|" << name << "|" << fixed
                 << setprecision(2) << 
                 (isDeposit ? balance + amount : balance - amount)
                 << endl;
        } else {
            temp << number << "|" << name << "|" << balance << endl;
        }
    }

    input.close();
    temp.close();

    remove("accounts.dat");
    rename("temp.dat", "accounts.dat");
}

void checkBalance() {
    int accNo;

    cout << "\nEnter Account Number: ";
    cin >> accNo;

    BankAccount account;

    if (findAccount(accNo, account)) {
        account.display();
    } else {
        cout << "Account not found!\n";
    }
}

void displayAllAccounts() {
    ifstream file("accounts.dat");

    if (!file.is_open()) {
        cout << "No accounts found.\n";
        return;
    }

    int number;
    string name;
    double balance;

    cout << "\n========== ALL ACCOUNTS ==========\n";

    while (file >> number) {
        file.ignore();
        getline(file, name, '|');
        file >> balance;
        file.ignore();

        cout << "\nAccount Number : " << number;
        cout << "\nCustomer Name  : " << name;
        cout << fixed << setprecision(2);
        cout << "\nBalance        : Rs. " << balance << endl;
    }

    file.close();
}

int main() {
    int choice;

    do {
        cout << "\n====================================";
        cout << "\n       BANK MANAGEMENT SYSTEM";
        cout << "\n====================================";
        cout << "\n1. Create Account";
        cout << "\n2. Deposit Money";
        cout << "\n3. Withdraw Money";
        cout << "\n4. Check Balance";
        cout << "\n5. Display All Accounts";
        cout << "\n6. Exit";
        cout << "\n====================================";
        cout << "\nEnter your choice: ";
        cin >> choice;

        switch (choice) {

        case 1: {
            BankAccount account;
            account.createAccount();

            // Check duplicate account number
            BankAccount existing;

            if (findAccount(account.getAccountNumber(), existing)) {
                cout << "Account number already exists!\n";
            } else {
                account.saveToFile();
                cout << "Account created successfully!\n";
            }

            break;
        }

        case 2: {
            int accNo;
            double amount;

            cout << "\nEnter Account Number: ";
            cin >> accNo;

            cout << "Enter Deposit Amount: Rs. ";
            cin >> amount;

            updateAccountBalance(accNo, amount, true);
            break;
        }

        case 3: {
            int accNo;
            double amount;

            cout << "\nEnter Account Number: ";
            cin >> accNo;

            cout << "Enter Withdrawal Amount: Rs. ";
            cin >> amount;

            updateAccountBalance(accNo, amount, false);
            break;
        }

        case 4:
            checkBalance();
            break;

        case 5:
            displayAllAccounts();
            break;

        case 6:
            cout << "\nThank you for using Bank Management System!\n";
            break;

        default:
            cout << "\nInvalid choice! Please try again.\n";
        }

    } while (choice != 6);

    return 0;
}
