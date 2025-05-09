import sys
import csv
from PyQt6.QtWidgets import (
    QApplication, QMainWindow, QStackedWidget, QWidget, QLabel,
    QLineEdit, QPushButton, QVBoxLayout, QMessageBox,
)
from PyQt6.QtCore import Qt
from PyQt6.QtGui import QFont

Account_File = "accounts.csv"

class ATM():
    def __init__(self):
        self.user = None #Tracks the logged-in user
        self.accounts = self.load_account() #loads account from csv or using the pin

    def load_account(self):
        try:
            with open(Account_File, "r") as f:
                reader = csv.DictReader(f)
                return {row["card_number"]: row for row in reader}
        except FileNotFoundError:
            return {
                "1234567890": {"card_number": "1234567890", "pin": "1234", "balance": "1000000"}
            }

    def save_account(self):
        with open(Account_File, "w", newline= "") as f:
            field_names = ["card_number", "pin", "balance"]
            writer = csv.DictWriter(f, field_names)
            writer.writeheader()
            for _ in self.accounts.values():
                writer.writerow(_)
    #The above saves account back to accounts.csv in CSV format

    def authenticate(self, card_number, pin):
        account = self.accounts.get(card_number)
        if account and account["pin"] == pin:
            return True
        return False
    #The above checks if the provided card_number and pin match any account
    #Then it sets the user on success

    def update_balance(self, amount):
        if self.user:
            self.user["balance"] = str(int(self.user["balance"]) + amount)
            #The above converts the balance stored as a string in the CSV to an integer
            self.save_account()
    #Updates the balance of the logged-in user and saves changes


class LanguageSelection(QWidget):
    def __init__(self, stacked_widget):
        super().__init__()
        self.stacked_widget = stacked_widget
        self.initUI()

    def initUI(self):
        layout = QVBoxLayout()
        self.setLayout(layout)

        title = QLabel("Choose Language / انتخاب زبان")
        title.setAlignment(Qt.AlignmentFlag.AlignCenter)
        title.setFont(QFont("Arial", 16))
        layout.addWidget(title)

        eng_btn = QPushButton("English")
        eng_btn.clicked.connect(lambda: self.switch_screen("english"))
        layout.addWidget(eng_btn)

        per_btn = QPushButton("Persian")
        per_btn.clicked.connect(lambda: self.switch_screen("persian"))
        layout.addWidget(per_btn)

    def switch_screen(self, language):
        self.stacked_widget.setCurrentIndex(1)
        self.stacked_widget.currentWidget().set_language(language)
    #Switch_screen ensures that in the atm app, the chosen language shows at the screen during transactions


class PIN(QWidget):
    def __init__(self, stacked_widget, atm):
        super().__init__()
        self.stacked_widget = stacked_widget
        #The stacked_widget is an instance that allow the PIN screen to switch to other screens
        self.atm = atm
        self.language = "english"
        self.initUI()
    #The above handles PIN input and authentication

    def initUI(self):
        self.layout = QVBoxLayout()
        self.setLayout(self.layout)

        self.title = QLabel()
        self.title.setAlignment(Qt.AlignmentFlag.AlignCenter)
        self.title.setFont(QFont("Arial", 16))

        self.pin = QLineEdit()
        self.pin.setEchoMode(QLineEdit.EchoMode.Password)
        self.pin.setMaxLength(4)

        self.submit_btn = QPushButton()
        self.submit_btn.clicked.connect(self.authenticate)

        self.layout.addWidget(self.title)
        self.layout.addWidget(QLabel("Enter your PIN / رمز خود را انتخاب کنید"))
        self.layout.addWidget(self.pin)
        self.layout.addWidget(self.submit_btn)

    def set_language(self, language):
        self.language = language
        self.update_text()

    def update_text(self):
        if self.language == "english":
            self.title.setText("Enter PIN")
            self.submit_btn.setText("Submit")
        else:
            self.title.setText("ورود رمز")
            self.submit_btn.setText("تایید")

    def authenticate(self):
        pin = self.pin.text()
        if self.atm.authenticate("1234567890", pin):
            self.stacked_widget.setCurrentIndex(2)
        else:
            QMessageBox.warning(self, "Error", "Invalid PIN / رمز نامعتبر")
            #The above warning shows an error message if PIN is invalid


class Menu(QWidget):
    def __init__(self, stacked_widget, atm):
        super().__init__()
        self.stacked_widget = stacked_widget #Allowing screen to switch to another screen
        self.atm = atm
        self.initUI()
    #Displays atm options

    def initUI(self):
        layout = QVBoxLayout()
        self.setLayout(layout)

        title = QLabel("Main Menu / منوی اصلی")
        title.setAlignment(Qt.AlignmentFlag.AlignCenter)
        layout.addWidget(title)

        withdraw_btn = QPushButton("Withdraw cash / برداشت وجه")
        withdraw_btn.clicked.connect(lambda: self.stacked_widget.setCurrentIndex(3))
        layout.addWidget(withdraw_btn)

        transfer_btn = QPushButton("Transfer Funds / انتقال وجه")
        transfer_btn.clicked.connect(lambda: self.stacked_widget.setCurrentIndex(4))
        layout.addWidget(transfer_btn)

        balance_btn = QPushButton("Check Balance / نمایش موجودی")
        balance_btn.clicked.connect(self.show_balance)
        layout.addWidget(balance_btn)

        change_pin_btn = QPushButton("Change PIN / تغییر رمز")
        change_pin_btn.clicked.connect(lambda: self.stacked_widget.setCurrentIndex(5))
        layout.addWidget(change_pin_btn)
    #The above is adding the necessary buttons for the ATM

    def show_balance(self):
        balance = self.atm.user["balance"]
        QMessageBox.information(self, "Balance", f"Your balance: {balance}")
    #Displays the user's balance


class ATMApp(QMainWindow):
    def __init__(self):
        super().__init__()
        self.atm= ATM()
        self.initUI()

    def initUI(self):
        self.stacked_widget = QStackedWidget()
        self.setCentralWidget(self.stacked_widget)

        self.language_window = LanguageSelection(self.stacked_widget)
        self.pin_window = PIN(self.stacked_widget, self.atm)
        self.menu_window = Menu(self.stacked_widget, self.atm)

        self.stacked_widget.addWidget(self.language_window)
        self.stacked_widget.addWidget(self.pin_window)
        self.stacked_widget.addWidget(self.menu_window)

        self.setWindowTitle("ATM Machine")
        self.setGeometry(300, 300, 400, 500)
        self.show()
    #The above is using QStackedWidget to manage multiple screens


if __name__ == "__main__":
    app = QApplication(sys.argv)
    ATM = ATMApp()
    sys.exit(app.exec())