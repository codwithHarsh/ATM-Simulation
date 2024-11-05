#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_TRANSACTIONS 10
#define MAX_USERS 4

// Transaction structure
typedef struct {
    char type[20];
    float amount;
    char details[50];
} Transaction;

// Account structure for each user
typedef struct {
    char username[50];
    char userPIN[5];
    float balance;
    Transaction transactions[MAX_TRANSACTIONS];
    int transactionCount;
} Account;

Account accounts[MAX_USERS] = {
    {"Harsh Wardhan Chaubey", "1234", 10000.00, {}, 0},
    {"Anshita Saikia", "5678", 15000.00, {}, 0},
    {"Harsimar Singh", "4321", 20000.00, {}, 0},
    {"Anjali", "8765", 25000.00, {}, 0}
};

Account* currentAccount = NULL; // Pointer to the logged-in account

// All the Function of ATM
void showMenu();
void balanceInquiry();
void cashWithdrawal();
void fundTransfer();
void deposit();
void miniStatement();
void changePIN();
int validatePIN(); // Change return type to int
void logTransaction(const char* type, float amount, const char* details);
void exitATM();
void delay(int seconds);

int main() {
    int choice;

    printf("Welcome to the ATM!\n");
    printf("Please insert your card (press Enter to continue)...\n");
    getchar();

    if (validatePIN() == 0) {
        printf("Incorrect PIN entered 3 times. Exiting...\n");
        exit(1);
    }

    while (1) {
        showMenu();
        printf("\nEnter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                balanceInquiry();
                break;
            case 2:
                cashWithdrawal();
                break;
            case 3:
                fundTransfer();
                break;
            case 4:
                deposit();
                break;
            case 5:
                miniStatement();
                break;
            case 6:
                changePIN();
                break;
            case 7:
                exitATM();
                break;
            default:
                printf("Invalid choice. Please try again.\n");
        }
    }

    return 0;
}

void showMenu() {
    printf("\n===== ATM MENU =====\n");
    printf("1. Balance Inquiry\n");
    printf("2. Cash Withdrawal\n");
    printf("3. Fund Transfer\n");
    printf("4. Deposit\n");
    printf("5. Mini Statement\n");
    printf("6. Change PIN\n");
    printf("7. Exit\n");
}

// Account PIN validation and user selection
int validatePIN() {
    char enteredPIN[5];
    int attempts = 0;

    while (attempts < 3) {
        printf("Enter your 4-digit PIN: ");
        scanf("%s", enteredPIN);

        for (int i = 0; i < MAX_USERS; i++) {
            if (strcmp(enteredPIN, accounts[i].userPIN) == 0) {
                currentAccount = &accounts[i];
                printf("Welcome, %s!\n", currentAccount->username);
                return 1; // Successfully logged in
            }
        }

        printf("Invalid PIN. Please try again.\n");
        attempts++;
    }

    return 0; // Failed after 3 attempts
}

// Balance inquiry
void balanceInquiry() {
    printf("\n===== BALANCE INQUIRY =====\n");
    printf("Account Holder: %s\n", currentAccount->username);
    printf("Available Balance: %.2f\n", currentAccount->balance);
}

// Cash withdrawal
void cashWithdrawal() {
    float amount;

    printf("\n===== CASH WITHDRAWAL =====\n");
    printf("Enter amount to withdraw: ");
    scanf("%f", &amount);

    if (amount > currentAccount->balance) {
        printf("Insufficient funds. Transaction failed.\n");
    } else if (amount <= 0) {
        printf("Invalid amount. Please enter a positive value.\n");
    } else {
        currentAccount->balance -= amount;
        printf("Please take your cash: %.2f\n", amount);
        printf("Transaction completed successfully.\nYour current balance is %.2f\n", currentAccount->balance);
        logTransaction("Withdrawal", amount, "Cash Withdrawal");
    }
}

// Fund transfer
void fundTransfer() {
    float amount;
    char destinationAccount[20];

    printf("\n===== FUND TRANSFER =====\n");
    printf("Enter destination account number: ");
    scanf("%s", destinationAccount);
    printf("Enter amount to transfer: ");
    scanf("%f", &amount);

    if (amount > currentAccount->balance) {
        printf("Insufficient funds. Transfer failed.\n");
    } else if (amount <= 0) {
        printf("Invalid amount. Please enter a positive value.\n");
    } else {
        currentAccount->balance -= amount;
        printf("Amount %.2f transferred to account %s successfully.\n", amount, destinationAccount);
        printf("Your current balance is: %.2f\n", currentAccount->balance);
        logTransaction("Transfer", amount, destinationAccount);
    }
}

// Deposit funds
void deposit() {
    float amount;

    printf("\n===== DEPOSIT =====\n");
    printf("Enter amount to deposit: ");
    scanf("%f", &amount);

    if (amount <= 0) {
        printf("Invalid amount. Please enter a positive value.\n");
    } else {
        currentAccount->balance += amount;
        printf("Amount %.2f deposited successfully.\n", amount);
        printf("Your new balance is: %.2f\n", currentAccount->balance);
        logTransaction("Deposit", amount, "Cash Deposit");
    }
}

// Mini statement (last 3 transactions)
void miniStatement() {
    printf("\n===== MINI STATEMENT =====\n");

    if (currentAccount->transactionCount == 0) {
        printf("No transactions to display.\n");
        return;
    }

    int start = currentAccount->transactionCount >= 3 ? currentAccount->transactionCount - 3 : 0;
    for (int i = start; i < currentAccount->transactionCount; i++) {
        printf("%d. %s of %.2f, Account Details: %s\n", i + 1, currentAccount->transactions[i].type,
               currentAccount->transactions[i].amount, currentAccount->transactions[i].details);
    }
}


// Change PIN
void changePIN() {
    char currentPIN[5];
    char newPIN[5];
    char confirmPIN[5];

    printf("\n===== CHANGE PIN =====\n");
    printf("Enter your current 4-digit PIN: ");
    scanf("%s", currentPIN);

    // Validate the current PIN
    if (strcmp(currentPIN, currentAccount->userPIN) != 0) {
        printf("Current PIN is incorrect. Please try again.\n");
        return; // Exit the function if the PIN is incorrect
    }

    printf("Enter new 4-digit PIN: ");
    scanf("%s", newPIN);
    printf("Confirm new PIN: ");
    scanf("%s", confirmPIN);

    if (strcmp(newPIN, confirmPIN) == 0) {
        // Update the user's PIN
        strcpy(currentAccount->userPIN, newPIN);
        printf("PIN changed successfully!\n");
    } else {
        printf("PIN mismatch. Please try again.\n");
    }
}


// Log transaction details
void logTransaction(const char* type, float amount, const char* details) {
    if (currentAccount->transactionCount < MAX_TRANSACTIONS) {
        strcpy(currentAccount->transactions[currentAccount->transactionCount].type, type);
        currentAccount->transactions[currentAccount->transactionCount].amount = amount;
        strcpy(currentAccount->transactions[currentAccount->transactionCount].details, details);
        currentAccount->transactionCount++;
    } else {
        for (int i = 1; i < MAX_TRANSACTIONS; i++) {
            currentAccount->transactions[i - 1] = currentAccount->transactions[i];
        }
        strcpy(currentAccount->transactions[MAX_TRANSACTIONS - 1].type, type);
        currentAccount->transactions[MAX_TRANSACTIONS - 1].amount = amount;
        strcpy(currentAccount->transactions[MAX_TRANSACTIONS - 1].details, details);
    }
}

// Exit ATM
void exitATM() {
    printf("Thank you for using the ATM, %s. Please take your card.\n", currentAccount->username);
    exit(0);
}

void delay(int seconds) {
    int milli_seconds = 1000 * seconds;
    clock_t start_time = clock();

    while (clock() < start_time + milli_seconds);
}
