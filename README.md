from tkinter import *
from tkinter import messagebox
from abc import ABC, abstractmethod

passwords = {
    "admin": "123"
}

class BankAccount:
    def __init__(self, balance=0.0):
        self._balance = balance

    def deposit(self, amount: float):
        self._balance += amount

    def withdraw(self, amount: float):
        if amount > self._balance:
            raise ValueError("Недостаточно средств")
        self._balance -= amount

    def get_balance(self):
        return self._balance

class AbstractWindow(ABC):
    def __init__(self, root=None):
        if root is None:
            self.root = Tk()
        else:
            self.root = Toplevel(root)

        self.root.geometry("500x500")
        self.root.title("BankSystem")

    @abstractmethod
    def _create_widgets(self):
        pass

    @abstractmethod
    def _pack_widgets(self):
        pass


class AbstractOperationWindow(AbstractWindow):
    def __init__(self, main_window, title):
        super().__init__(main_window.root)
        self.main_window = main_window
        self.account = main_window.account

        self.root.title(title)
        self.root.protocol("WM_DELETE_WINDOW", self._close)

        self._create_widgets()
        self._pack_widgets()

    def _close(self):
        self.destroy_reference()
        self.root.destroy()

    @abstractmethod
    def destroy_reference(self):
        pass

    @abstractmethod
    def perform_operation(self, amount: float):
        pass

    def _handle_action(self):
        try:
            amount = float(self.input.get())

            self.perform_operation(amount)

            self.label.config(text=f"Баланс: {self.account.get_balance()}")
            self.input.delete(0, END)

        except ValueError as e:
            messagebox.showerror("Ошибка", str(e))

    def _create_widgets(self):
        self.label = Label(self.root, text=f"Баланс: {self.account.get_balance()}", font=("Arial", 16))
        self.input = Entry(self.root)
        self.button = Button(self.root, command=self._handle_action)

    def _pack_widgets(self):
        self.label.pack(pady=20)
        self.input.pack(pady=10)
        self.button.pack(pady=10)


class LoginWindow(AbstractWindow):

    def __init__(self):
        super().__init__()
        self.root.title("Вход")

        self._create_widgets()
        self._pack_widgets()

    def __enter(self):
        login = self.input_login.get().strip()
        password = self.input_password.get().strip()

        if login in passwords and passwords[login] == password:
            self.root.withdraw()
            account = BankAccount()
            MainWindow(login, account, self.root)
        else:
            messagebox.showerror("Ошибка", "Неверные данные")

    def _create_widgets(self):
        self.label_login = Label(self.root, text="Логин", font=("Arial", 16))
        self.input_login = Entry(self.root)

        self.label_password = Label(self.root, text="Пароль", font=("Arial", 16))
        self.input_password = Entry(self.root, show="*")

        self.button_login = Button(self.root, text="Войти", command=self.__enter)

    def _pack_widgets(self):
        self.label_login.pack(pady=10)
        self.input_login.pack(pady=10)
        self.label_password.pack(pady=10)
        self.input_password.pack(pady=10)
        self.button_login.pack(pady=20)


class MainWindow(AbstractWindow):

    def __init__(self, login, account, root):
        super().__init__(root)
        self.account = account
        self._login = login

        self.deposit_window = None
        self.withdraw_window = None

        self.root.title("Главное окно")

        self._create_widgets()
        self._pack_widgets()

    def __get_balance(self):
        messagebox.showinfo("Баланс", f"{self.account.get_balance()}")

    def __open_deposit(self):
        if self.deposit_window is None:
            self.deposit_window = DepositWindow(self)

    def __open_withdraw(self):
        if self.withdraw_window is None:
            self.withdraw_window = WithdrawWindow(self)

    def _create_widgets(self):
        self.label = Label(self.root, text=f"Привет, {self._login}", font=("Arial", 16))
        self.btn_balance = Button(self.root, text="Баланс", command=self.__get_balance)
        self.btn_deposit = Button(self.root, text="Пополнить", command=self.__open_deposit)
        self.btn_withdraw = Button(self.root, text="Снять", command=self.__open_withdraw)

    def _pack_widgets(self):
        self.label.pack(pady=20)
        self.btn_balance.pack(pady=10)
        self.btn_deposit.pack(pady=10)
        self.btn_withdraw.pack(pady=10)


class DepositWindow(AbstractOperationWindow):

    def __init__(self, main_window):
        super().__init__(main_window, "Пополнение")
        self.button.config(text="Пополнить")

    def destroy_reference(self):
        self.main_window.deposit_window = None

    def perform_operation(self, amount: float):
        self.account.deposit(amount)
        messagebox.showinfo("Успех", f"+{amount}")


class WithdrawWindow(AbstractOperationWindow):

    def __init__(self, main_window):
        super().__init__(main_window, "Снятие")
        self.button.config(text="Снять")

    def destroy_reference(self):
        self.main_window.withdraw_window = None

    def perform_operation(self, amount: float):
        self.account.withdraw(amount)
        messagebox.showinfo("Успех", f"-{amount}")


if __name__ == "__main__":
    app = LoginWindow()
    app.root.mainloop()
