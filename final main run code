import tkinter as tk
from tkinter import messagebox
from PIL import Image, ImageTk
import os

from database import save_order

# ===========================
# PATHS & CONSTANTS
# ===========================

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
# Put your food images in a folder called "FoodImages" in the same location as this .py file
IMG_DIR = os.path.join(BASE_DIR, "FoodImages")
TAX_RATE = 0.15  # 15% GST (you can change if you want)


# ===========================
# MODELS
# ===========================

class FoodItem:
    def __init__(self, product_id, name, price, image_path, stock):
        self.id = product_id          # matches product_id in MySQL
        self.name = name
        self.price = price
        self.image_path = image_path
        self.stock = stock  # current stock level


class CartItem:
    def __init__(self, product: FoodItem, quantity: int):
        self.product = product
        self.quantity = quantity


FOODS = [
    FoodItem(1, "Margherita Pizza", 12.99, os.path.join(IMG_DIR, "pizza.png"), 10),
    FoodItem(2, "Cheese Burger",    10.49, os.path.join(IMG_DIR, "cheese_burger.png"), 8),
    FoodItem(3, "Veggie Burger",     9.99, os.path.join(IMG_DIR, "veggie_burger.png"), 7),
    FoodItem(4, "French Fries",      4.99, os.path.join(IMG_DIR, "fries.png"), 20),
    FoodItem(5, "Chicken Wings",    11.99, os.path.join(IMG_DIR, "wings.png"), 12),
    FoodItem(6, "Pasta Alfredo",    13.49, os.path.join(IMG_DIR, "pasta.png"), 9),
    FoodItem(7, "Caesar Salad",      8.99, os.path.join(IMG_DIR, "salad.png"), 6),
    FoodItem(8, "Chocolate Brownie", 6.49, os.path.join(IMG_DIR, "brownie.png"), 15),
]

VALID_USERNAME = "staff"
VALID_PASSWORD = "1234"


# ===========================
# MAIN APP
# ===========================

class SmartBitesApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("SmartBites - Restaurant POS")
        self.geometry("1200x700")
        self.config(bg="#f7f7f7")

        self.current_user = None  # store logged-in username

        # inventory & cart
        self.food_items = FOODS
        self.cart_items = []      # list of CartItem
        self.discount_choice = "No Discount"

        self.current_frame = None
        self.show_login()

    # ------------- frame switching -------------

    def show_login(self):
        if self.current_frame:
            self.current_frame.destroy()
        self.current_frame = LoginFrame(self)
        self.current_frame.pack(fill="both", expand=True)

    def show_dashboard(self):
        if self.current_frame:
            self.current_frame.destroy()
        self.current_frame = DashboardFrame(self)
        self.current_frame.pack(fill="both", expand=True)

    # ------------- cart helpers -------------

    def add_to_cart(self, product: FoodItem, quantity: int = 1):
        """Add a food item with quantity to the cart, checking stock."""
        try:
            quantity = int(quantity)
        except ValueError:
            quantity = 1

        if quantity <= 0:
            messagebox.showwarning("Quantity", "Quantity must be at least 1.")
            return

        already_in_cart = sum(item.quantity for item in self.cart_items if item.product == product)
        if already_in_cart + quantity > product.stock:
            available = max(product.stock - already_in_cart, 0)
            messagebox.showwarning(
                "Stock",
                f"Not enough stock for {product.name}.\n"
                f"Available to add: {available}"
            )
            return

        for item in self.cart_items:
            if item.product == product:
                item.quantity += quantity
                break
        else:
            self.cart_items.append(CartItem(product, quantity))

        if isinstance(self.current_frame, DashboardFrame):
            self.current_frame.update_cart_panel()

    def remove_from_cart(self, index: int):
        if 0 <= index < len(self.cart_items):
            del self.cart_items[index]
        if isinstance(self.current_frame, DashboardFrame):
            self.current_frame.update_cart_panel()

    def clear_cart(self):
        self.cart_items.clear()
        if isinstance(self.current_frame, DashboardFrame):
            self.current_frame.update_cart_panel()

    def increase_cart_quantity(self, index: int):
        if 0 <= index < len(self.cart_items):
            item = self.cart_items[index]
            if item.quantity < item.product.stock:
                item.quantity += 1
            else:
                messagebox.showwarning("Stock", "No more stock available for this item.")
        if isinstance(self.current_frame, DashboardFrame):
            self.current_frame.update_cart_panel()

    def decrease_cart_quantity(self, index: int):
        if 0 <= index < len(self.cart_items):
            item = self.cart_items[index]
            if item.quantity > 1:
                item.quantity -= 1
            else:
                # if quantity is 1 and user presses "-", remove item
                self.remove_from_cart(index)
        if isinstance(self.current_frame, DashboardFrame):
            self.current_frame.update_cart_panel()


# ===========================
# LOGIN FRAME
# ===========================

class LoginFrame(tk.Frame):
    def __init__(self, master: SmartBitesApp):
        super().__init__(master, bg="#ffffff")

        container = tk.Frame(self, bg="#ffffff")
        container.place(relx=0.5, rely=0.5, anchor="center")

        # Logo / Title
        title = tk.Label(
            container,
            text="SmartBites Restaurant POS\nStaff Login",
            font=("Segoe UI", 22, "bold"),
            bg="#ffffff",
            fg="#333333",
            justify="center"
        )
        title.pack(pady=(0, 20))

        form = tk.Frame(container, bg="#ffffff")
        form.pack()

        tk.Label(form, text="Username", bg="#ffffff",
                 fg="#555555", font=("Segoe UI", 11)).grid(row=0, column=0, sticky="e", padx=5, pady=5)
        tk.Label(form, text="Password", bg="#ffffff",
                 fg="#555555", font=("Segoe UI", 11)).grid(row=1, column=0, sticky="e", padx=5, pady=5)

        self.username_entry = tk.Entry(form, font=("Segoe UI", 11), width=25)
        self.password_entry = tk.Entry(form, font=("Segoe UI", 11),
                                       show="*", width=25)
        self.username_entry.grid(row=0, column=1, padx=5, pady=5)
        self.password_entry.grid(row=1, column=1, padx=5, pady=5)

        login_btn = tk.Button(
            container,
            text="Login",
            font=("Segoe UI", 11, "bold"),
            width=18,
            pady=6,
            command=self.handle_login,
            bg="#ff7043",
            fg="white",
            activebackground="#ff8a65",
            bd=0,
            cursor="hand2"
        )
        login_btn.pack(pady=18)

        note = tk.Label(
            container,
            text='Demo → username: "staff"  password: "1234"',
            font=("Segoe UI", 9),
            bg="#ffffff",
            fg="#888888"
        )
        note.pack()

    def handle_login(self):
        user = self.username_entry.get().strip()
        pwd = self.password_entry.get().strip()

        if user == VALID_USERNAME and pwd == VALID_PASSWORD:
            self.master.current_user = user
            self.master.show_dashboard()
        else:
            messagebox.showerror("Login Failed", "Invalid username or password.")


# ===========================
# DASHBOARD FRAME
# ===========================

class DashboardFrame(tk.Frame):
    def __init__(self, master: SmartBitesApp):
        super().__init__(master, bg="#f7f7f7")

        # TOP BAR
        top_bar = tk.Frame(self, bg="#ffffff", height=80)
        top_bar.pack(side="top", fill="x")

        title = tk.Label(
            top_bar,
            text="SmartBites - Food Dashboard",
            font=("Segoe UI", 22, "bold"),
            bg="#ffffff",
            fg="#333333"
        )
        title.pack(side="left", padx=20)

        # Logged in user
        user_lbl = tk.Label(
            top_bar,
            text=f"Logged in as: {self.master.current_user}",
            font=("Segoe UI", 10),
            bg="#ffffff",
            fg="#555555"
        )
        user_lbl.pack(side="right", padx=10)

        # Logout Button
        logout_btn = tk.Button(
            top_bar,
            text="Logout",
            font=("Segoe UI", 10, "bold"),
            command=self.master.show_login,
            bg="#333333",
            fg="white",
            activebackground="#555555",
            bd=0,
            padx=15,
            pady=6,
            cursor="hand2"
        )
        logout_btn.pack(side="right", padx=20)

        # MAIN CONTENT
        content = tk.Frame(self, bg="#f7f7f7")
        content.pack(fill="both", expand=True, padx=15, pady=15)

        # LEFT SIDE: FOOD GRID
        product_frame = tk.Frame(content, bg="#f7f7f7")
        product_frame.pack(side="left", fill="both", expand=True)

        canvas = tk.Canvas(product_frame, bg="#f7f7f7", highlightthickness=0)
        canvas.pack(side="left", fill="both", expand=True)

        scrollbar = tk.Scrollbar(product_frame, orient="vertical",
                                 command=canvas.yview)
        scrollbar.pack(side="right", fill="y")

        canvas.configure(yscrollcommand=scrollbar.set)

        self.inner_product_frame = tk.Frame(canvas, bg="#f7f7f7")
        canvas.create_window((0, 0), window=self.inner_product_frame, anchor="nw")

        self.inner_product_frame.bind(
            "<Configure>",
            lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
        )

        self.product_photos = []   # keep references
        self.product_stock_labels = {}  # FoodItem -> Label
        self.build_food_cards()

        # RIGHT SIDE: CART PANEL
        self.cart_panel = tk.Frame(content, bg="#ffffff", width=350)
        self.cart_panel.pack(side="right", fill="y", padx=(10, 0))
        self.cart_panel.pack_propagate(False)

        self.build_cart_panel()

    # ------------------
    # FOOD CARDS
    # ------------------
    def build_food_cards(self):
        row = 0
        col = 0
        for food in self.master.food_items:
            card = tk.Frame(
                self.inner_product_frame,
                bg="#ffffff",
                highlightbackground="#e3e3e3",
                highlightthickness=1,
                padx=10,
                pady=10
            )
            card.grid(row=row, column=col, padx=10, pady=10, sticky="nsew")

            # Food Image
            img_label = tk.Label(card, bg="#ffffff")
            img_label.pack()

            try:
                img = Image.open(food.image_path)
                img = img.resize((140, 140), Image.LANCZOS)
                tk_img = ImageTk.PhotoImage(img)
                self.product_photos.append(tk_img)
                img_label.config(image=tk_img)
            except Exception:
                img_label.config(
                    text="No Image",
                    font=("Segoe UI", 10),
                    fg="#aaaaaa",
                    width=18,
                    height=8
                )

            # Food name
            name_lbl = tk.Label(
                card,
                text=food.name,
                font=("Segoe UI", 14, "bold"),
                bg="#ffffff",
                fg="#333333",
                wraplength=180,
                justify="center"
            )
            name_lbl.pack(pady=(5, 0))

            # Price
            price_lbl = tk.Label(
                card,
                text=f"${food.price:.2f}",
                font=("Segoe UI", 12),
                bg="#ffffff",
                fg="#ff7043"
            )
            price_lbl.pack()

            # Stock label
            stock_color = "#2e7d32" if food.stock > 2 else "#b71c1c"
            stock_lbl = tk.Label(
                card,
                text=f"Stock: {food.stock}",
                font=("Segoe UI", 9),
                bg="#ffffff",
                fg=stock_color
            )
            stock_lbl.pack()
            self.product_stock_labels[food] = stock_lbl

            # Quantity selector
            qty_frame = tk.Frame(card, bg="#ffffff")
            qty_frame.pack(pady=(5, 0))

            tk.Label(qty_frame, text="Qty:", bg="#ffffff",
                     fg="#555555", font=("Segoe UI", 9)).pack(side="left")

            qty_var = tk.IntVar(value=1)
            qty_spin = tk.Spinbox(
                qty_frame,
                from_=1,
                to=max(food.stock, 1),
                width=5,
                textvariable=qty_var,
                font=("Segoe UI", 9)
            )
            qty_spin.pack(side="left", padx=5)

            # Add to cart
            add_btn = tk.Button(
                card,
                text="Add to Cart",
                font=("Segoe UI", 10, "bold"),
                bg="#ff7043",
                fg="white",
                activebackground="#ff8a65",
                bd=0,
                padx=10,
                pady=4,
                cursor="hand2",
                command=lambda f=food, q=qty_var: self.master.add_to_cart(f, q.get())
            )
            add_btn.pack(pady=(10, 0))

            col += 1
            if col >= 3:
                col = 0
                row += 1

    def refresh_stock_labels(self):
        for food, label in self.product_stock_labels.items():
            stock_color = "#2e7d32" if food.stock > 2 else "#b71c1c"
            label.config(text=f"Stock: {food.stock}", fg=stock_color)

    # ------------------
    # CART PANEL
    # ------------------
    def build_cart_panel(self):
        header = tk.Label(
            self.cart_panel,
            text="Order Cart",
            font=("Segoe UI", 16, "bold"),
            bg="#ffffff",
            fg="#333333"
        )
        header.pack(anchor="w", padx=10, pady=(10, 5))

        # Discount dropdown
        discount_frame = tk.Frame(self.cart_panel, bg="#ffffff")
        discount_frame.pack(fill="x", padx=10, pady=(0, 10))

        tk.Label(
            discount_frame,
            text="Customer Type:",
            bg="#ffffff",
            fg="#555555",
            font=("Segoe UI", 10)
        ).pack(side="left")

        self.discount_var = tk.StringVar(value=self.master.discount_choice)
        discount_options = [
            "No Discount",
            "Regular Member (5%)",
            "Student (10%)",
            "VIP (15%)"
        ]
        discount_menu = tk.OptionMenu(
            discount_frame,
            self.discount_var,
            *discount_options,
            command=lambda _: self.update_cart_panel()
        )
        discount_menu.config(font=("Segoe UI", 9), bg="#f0f0f0", bd=0)
        discount_menu.pack(side="left", padx=5)

        # list area
        self.cart_list_frame = tk.Frame(self.cart_panel, bg="#ffffff")
        self.cart_list_frame.pack(fill="both", expand=True, padx=10)

        # totals
        self.subtotal_label = tk.Label(
            self.cart_panel,
            text="Sub Total: $0.00",
            font=("Segoe UI", 10),
            bg="#ffffff",
            fg="#555555"
        )
        self.subtotal_label.pack(anchor="w", padx=10)

        self.discount_label = tk.Label(
            self.cart_panel,
            text="Discount: $0.00",
            font=("Segoe UI", 10),
            bg="#ffffff",
            fg="#555555"
        )
        self.discount_label.pack(anchor="w", padx=10)

        self.tax_label = tk.Label(
            self.cart_panel,
            text="Tax: $0.00",
            font=("Segoe UI", 10),
            bg="#ffffff",
            fg="#555555"
        )
        self.tax_label.pack(anchor="w", padx=10)

        self.total_label = tk.Label(
            self.cart_panel,
            text="Total: $0.00",
            font=("Segoe UI", 12, "bold"),
            bg="#ffffff",
            fg="#ff7043"
        )
        self.total_label.pack(anchor="w", padx=10, pady=(0, 5))

        btn_frame = tk.Frame(self.cart_panel, bg="#ffffff")
        btn_frame.pack(fill="x", padx=10, pady=(5, 10))

        clear_btn = tk.Button(
            btn_frame,
            text="Clear",
            font=("Segoe UI", 9, "bold"),
            command=self.master.clear_cart,
            bg="#f0f0f0",
            fg="#333333",
            bd=0,
            padx=10,
            pady=4,
            cursor="hand2"
        )
        clear_btn.pack(side="left")

        pay_btn = tk.Button(
            btn_frame,
            text="Go to Payment",
            font=("Segoe UI", 9, "bold"),
            command=self.open_payment_window,
            bg="#333333",
            fg="white",
            bd=0,
            padx=10,
            pady=4,
            cursor="hand2"
        )
        pay_btn.pack(side="right")

        self.update_cart_panel()

    def get_discount_rate(self):
        choice = self.discount_var.get()
        if choice.startswith("Regular"):
            return 0.05
        elif choice.startswith("Student"):
            return 0.10
        elif choice.startswith("VIP"):
            return 0.15
        return 0.0

    def calculate_totals(self):
        subtotal = 0.0
        for item in self.master.cart_items:
            subtotal += item.product.price * item.quantity

        rate = self.get_discount_rate()
        discount_amount = subtotal * rate
        taxable_amount = max(subtotal - discount_amount, 0)
        tax_amount = taxable_amount * TAX_RATE
        total = taxable_amount + tax_amount
        return subtotal, rate, discount_amount, tax_amount, total

    def update_cart_panel(self):
        # Clear list
        for w in self.cart_list_frame.winfo_children():
            w.destroy()

        # create rows
        for idx, cart_item in enumerate(self.master.cart_items):
            product = cart_item.product
            row = tk.Frame(self.cart_list_frame, bg="#ffffff")
            row.pack(fill="x", pady=2)

            name_lbl = tk.Label(
                row,
                text=f"{idx+1}. {product.name}",
                bg="#ffffff",
                fg="#555555",
                font=("Segoe UI", 9),
                wraplength=160,
                justify="left"
            )
            name_lbl.pack(side="left")

            # Quantity controls
            qty_frame = tk.Frame(row, bg="#ffffff")
            qty_frame.pack(side="left", padx=5)

            minus_btn = tk.Button(
                qty_frame,
                text="-",
                font=("Segoe UI", 8, "bold"),
                bg="#f0f0f0",
                fg="#333333",
                bd=0,
                width=2,
                cursor="hand2",
                command=lambda i=idx: self.master.decrease_cart_quantity(i)
            )
            minus_btn.pack(side="left")

            qty_lbl = tk.Label(
                qty_frame,
                text=str(cart_item.quantity),
                bg="#ffffff",
                fg="#555555",
                font=("Segoe UI", 9)
            )
            qty_lbl.pack(side="left", padx=3)

            plus_btn = tk.Button(
                qty_frame,
                text="+",
                font=("Segoe UI", 8, "bold"),
                bg="#f0f0f0",
                fg="#333333",
                bd=0,
                width=2,
                cursor="hand2",
                command=lambda i=idx: self.master.increase_cart_quantity(i)
            )
            plus_btn.pack(side="left")

            # line total
            line_total = cart_item.product.price * cart_item.quantity
            price_lbl = tk.Label(
                row,
                text=f"${line_total:.2f}",
                bg="#ffffff",
                fg="#ff7043",
                font=("Segoe UI", 9)
            )
            price_lbl.pack(side="right")

            # Remove button
            remove_btn = tk.Button(
                row,
                text="X",
                font=("Segoe UI", 8, "bold"),
                bg="#f0f0f0",
                fg="#333333",
                bd=0,
                padx=4,
                cursor="hand2",
                command=lambda i=idx: self.master.remove_from_cart(i)
            )
            remove_btn.pack(side="right", padx=5)

        subtotal, rate, discount_amount, tax_amount, total = self.calculate_totals()

        self.master.discount_choice = self.discount_var.get()

        self.subtotal_label.config(text=f"Sub Total: ${subtotal:.2f}")
        self.discount_label.config(
            text=f"Discount ({int(rate*100)}%): -${discount_amount:.2f}")
        self.tax_label.config(
            text=f"Tax ({int(TAX_RATE*100)}%): +${tax_amount:.2f}")
        self.total_label.config(text=f"Total: ${total:.2f}")

    # ------------------
    # PAYMENT WINDOW
    # ------------------
        # ------------------
    # PAYMENT WINDOW
    # ------------------
    def open_payment_window(self):
        if not self.master.cart_items:
            messagebox.showwarning("Cart Empty", "Please add some items first.")
            return

        # calculate totals once for this payment
        subtotal, rate, discount_amount, tax_amount, total = self.calculate_totals()

        # Create Payment Window
        pay_win = tk.Toplevel(self)
        pay_win.title("SmartBites - Payment")
        pay_win.geometry("520x600")
        pay_win.config(bg="#ffffff")

        # Header
        tk.Label(
            pay_win,
            text="💳 Secure Payment",
            font=("Segoe UI", 20, "bold"),
            bg="#ffffff",
            fg="#333333"
        ).pack(pady=15)

        # Order Summary Box
        sum_frame = tk.Frame(pay_win, bg="#f7f7f7",
                             highlightbackground="#dddddd", highlightthickness=1)
        sum_frame.pack(fill="x", padx=20, pady=10)

        tk.Label(sum_frame, text="Order Summary", font=("Segoe UI", 12, "bold"),
                 bg="#f7f7f7").pack(anchor="w", padx=10, pady=(10, 5))

        tk.Label(sum_frame, text=f"Subtotal: ${subtotal:.2f}", bg="#f7f7f7").pack(anchor="w", padx=10)
        tk.Label(sum_frame, text=f"Discount ({int(rate*100)}%): -${discount_amount:.2f}",
                 bg="#f7f7f7").pack(anchor="w", padx=10)
        tk.Label(sum_frame, text=f"Tax ({int(TAX_RATE*100)}%): +${tax_amount:.2f}",
                 bg="#f7f7f7").pack(anchor="w", padx=10)
        tk.Label(sum_frame, text=f"Total: ${total:.2f}", font=("Segoe UI", 11, "bold"),
                 fg="#ff7043", bg="#f7f7f7").pack(anchor="w", padx=10, pady=(0, 10))

        # Payment Form
        form = tk.Frame(pay_win, bg="#ffffff")
        form.pack(pady=5)

        # Payment method (Card / Cash)
        payment_method_var = tk.StringVar(value="Card")
        tk.Label(form, text="Payment Method", bg="#ffffff", fg="#555555",
                 font=("Segoe UI", 10)).grid(row=0, column=0, sticky="e", padx=10, pady=8)
        pm_frame = tk.Frame(form, bg="#ffffff")
        pm_frame.grid(row=0, column=1, sticky="w")

        tk.Radiobutton(
            pm_frame, text="Card", variable=payment_method_var, value="Card",
            bg="#ffffff", fg="#555555", font=("Segoe UI", 9), anchor="w"
        ).pack(side="left")
        tk.Radiobutton(
            pm_frame, text="Cash", variable=payment_method_var, value="Cash",
            bg="#ffffff", fg="#555555", font=("Segoe UI", 9), anchor="w"
        ).pack(side="left", padx=10)

        # Card Brand Label (for card numbers)
        self.card_brand = tk.Label(form, text="", font=("Segoe UI", 9, "bold"),
                                   bg="#ffffff", fg="#555555")
        self.card_brand.grid(row=0, column=2, padx=5)

        # Card details
        tk.Label(form, text="Card Number", bg="#ffffff", fg="#555555",
                 font=("Segoe UI", 10)).grid(row=1, column=0, sticky="e", padx=10, pady=8)
        tk.Label(form, text="Name on Card", bg="#ffffff", fg="#555555",
                 font=("Segoe UI", 10)).grid(row=2, column=0, sticky="e", padx=10, pady=8)
        tk.Label(form, text="Expiry (MM/YY)", bg="#ffffff", fg="#555555",
                 font=("Segoe UI", 10)).grid(row=3, column=0, sticky="e", padx=10, pady=8)
        tk.Label(form, text="CVV", bg="#ffffff", fg="#555555",
                 font=("Segoe UI", 10)).grid(row=4, column=0, sticky="e", padx=10, pady=8)

        card_entry = tk.Entry(form, width=25, font=("Segoe UI", 10))
        name_entry = tk.Entry(form, width=25, font=("Segoe UI", 10))
        exp_entry = tk.Entry(form, width=8, font=("Segoe UI", 10))
        cvv_entry = tk.Entry(form, width=5, font=("Segoe UI", 10), show="*")

        card_entry.grid(row=1, column=1)
        name_entry.grid(row=2, column=1)
        exp_entry.grid(row=3, column=1)
        cvv_entry.grid(row=4, column=1)

        # Cash given (for cash payment)
        tk.Label(form, text="Cash Given ($)", bg="#ffffff", fg="#555555",
                 font=("Segoe UI", 10)).grid(row=5, column=0, sticky="e", padx=10, pady=8)
        cash_entry = tk.Entry(form, width=10, font=("Segoe UI", 10))
        cash_entry.grid(row=5, column=1, sticky="w")

        change_label = tk.Label(form, text="", bg="#ffffff", fg="#2e7d32",
                                font=("Segoe UI", 9))
        change_label.grid(row=6, column=1, sticky="w", pady=(0, 5))

        # Auto card detection
        def detect_card_brand(card_number):
            if card_number.startswith("4"):
                self.card_brand.config(text="VISA")
            elif card_number.startswith("5"):
                self.card_brand.config(text="MasterCard")
            else:
                self.card_brand.config(text="")

        def on_card_change(event):
            detect_card_brand(card_entry.get())
        card_entry.bind("<KeyRelease>", on_card_change)

        # When user types cash, show change (only if enough)
        def on_cash_change(event):
            text = cash_entry.get().strip()
            if not text:
                change_label.config(text="")
                return
            try:
                amount = float(text)
            except ValueError:
                change_label.config(text="Enter number only.")
                return
            if amount < total:
                change_label.config(text="Not enough cash.")
            else:
                change = amount - total
                change_label.config(text=f"Change to return: ${change:.2f}")

        cash_entry.bind("<KeyRelease>", on_cash_change)

        # Confirm Payment Logic
        def confirm_payment():
            payment_method = payment_method_var.get()

            # validation
            cash_given = None
            change = 0.0

            if payment_method == "Card":
                card = card_entry.get().strip()
                name = name_entry.get().strip()
                exp = exp_entry.get().strip()
                cvv = cvv_entry.get().strip()

                if len(card) < 12 or not card.isdigit():
                    messagebox.showerror("Invalid", "Enter a valid card number.")
                    return
                if not name:
                    messagebox.showerror("Invalid", "Enter the name on card.")
                    return
                if "/" not in exp or len(exp) != 5:
                    messagebox.showerror("Invalid", "Expiry must be MM/YY format.")
                    return
                if len(cvv) != 3 or not cvv.isdigit():
                    messagebox.showerror("Invalid", "CVV must be 3 digits.")
                    return

            elif payment_method == "Cash":
                text = cash_entry.get().strip()
                if not text:
                    messagebox.showerror("Invalid", "Enter the cash given by customer.")
                    return
                try:
                    cash_given = float(text)
                except ValueError:
                    messagebox.showerror("Invalid", "Cash must be a number.")
                    return
                if cash_given < total:
                    messagebox.showerror(
                        "Invalid",
                        "Cash is less than total amount."
                    )
                    return
                change = cash_given - total

            # ---------- SAVE ORDER TO DATABASE ----------
            try:
                save_order(
                    staff_username=self.master.current_user,
                    customer_type=self.discount_var.get(),
                    subtotal=subtotal,
                    discount=discount_amount,
                    tax=tax_amount,
                    total=total,
                    payment_method=payment_method,
                    cart_items=self.master.cart_items
                )
            except Exception as e:
                print("Could not save order to DB:", e)
            # ------------------------------------------------

            # Update inventory (reduce stock)
            for cart_item in self.master.cart_items:
                cart_item.product.stock -= cart_item.quantity
                if cart_item.product.stock < 0:
                    cart_item.product.stock = 0

            self.refresh_stock_labels()

            # RECEIPT WINDOW
            receipt = tk.Toplevel(pay_win)
            receipt.title("SmartBites - Receipt")
            receipt.geometry("480x520")
            receipt.config(bg="#ffffff")

            tk.Label(
                receipt,
                text="🧾 SmartBites Restaurant",
                font=("Segoe UI", 18, "bold"),
                bg="#ffffff",
                fg="#333333"
            ).pack(pady=15)

            rframe = tk.Frame(receipt, bg="#ffffff")
            rframe.pack(fill="both", expand=True, padx=20)

            for cart_item in self.master.cart_items:
                p = cart_item.product
                row = tk.Frame(rframe, bg="#ffffff")
                row.pack(fill="x", pady=2)
                tk.Label(row, text=f"{p.name} x{cart_item.quantity}", bg="#ffffff",
                         fg="#555555").pack(side="left")
                line_total = p.price * cart_item.quantity
                tk.Label(row, text=f"${line_total:.2f}", bg="#ffffff",
                         fg="#ff7043").pack(side="right")

            tk.Label(receipt, text=f"Subtotal: ${subtotal:.2f}", bg="#ffffff").pack(pady=(10, 0))
            tk.Label(receipt, text=f"Discount: -${discount_amount:.2f}", bg="#ffffff").pack()
            tk.Label(receipt, text=f"Tax: +${tax_amount:.2f}", bg="#ffffff").pack()
            tk.Label(receipt, text=f"Grand Total: ${total:.2f}",
                     font=("Segoe UI", 12, "bold"), fg="#ff7043", bg="#ffffff").pack(pady=(0, 5))

            tk.Label(
                receipt,
                text=f"Payment Method: {payment_method}",
                bg="#ffffff",
                fg="#555555",
                font=("Segoe UI", 10)
            ).pack()

            if payment_method == "Cash":
                tk.Label(
                    receipt,
                    text=f"Cash Given: ${cash_given:.2f}",
                    bg="#ffffff",
                    fg="#555555",
                    font=("Segoe UI", 10)
                ).pack()
                tk.Label(
                    receipt,
                    text=f"Change Returned: ${change:.2f}",
                    bg="#ffffff",
                    fg="#555555",
                    font=("Segoe UI", 10)
                ).pack()

            tk.Label(
                receipt,
                text=f"Processed by: {self.master.current_user}",
                bg="#ffffff",
                fg="#555555",
                font=("Segoe UI", 10)
            ).pack(pady=(5, 0))

            tk.Label(
                receipt,
                text="Thank you for dining with us!",
                bg="#ffffff",
                fg="#2e7d32",
                font=("Segoe UI", 11, "bold")
            ).pack(pady=10)

            tk.Button(
                receipt,
                text="Close",
                font=("Segoe UI", 10, "bold"),
                bg="#333333",
                fg="white",
                bd=0,
                padx=14,
                pady=6,
                cursor="hand2",
                command=receipt.destroy
            ).pack(pady=10)

            messagebox.showinfo("Success", "Payment completed successfully!")
            self.master.clear_cart()
            pay_win.destroy()

        # Payment Button
        tk.Button(
            pay_win,
            text="Confirm Payment",
            font=("Segoe UI", 12, "bold"),
            bg="#ff7043",
            fg="white",
            bd=0,
            padx=20,
            pady=10,
            cursor="hand2",
            command=confirm_payment
        ).pack(pady=25)


# ===========================
# RUN
# ===========================

if __name__ == "__main__":
    app = SmartBitesApp()
    app.mainloop()


