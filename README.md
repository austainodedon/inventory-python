# inventory-python
# a python inventory for supermarkets
from tkinter import *
from tkinter import ttk
import sqlite3

class ProductDB :

    # Will hold database connection
    db_conn = 0
    # A cursor is used to traverse the records of a result
    theCursor = 0
    # Will store the current student selected
    curr_student = 0

    def setup_db(self):

        # Open or create database
        self.db_conn = sqlite3.connect('product.db')

        # The cursor traverses the records
        self.theCursor = self.db_conn.cursor()

        # Create the table if it doesn't exist
        try:
            self.db_conn.execute("CREATE TABLE if not exists Products(ID INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, PNum TEXT NOT NULL, GName TEXT NOT NULL, PDesc TEXT NOT NULL, SQuant INTEGER NOT NULL);")

            self.db_conn.commit()

        except sqlite3.OperationalError:
            print("ERROR : Table not created")

    def prod_submit(self):

        # Insert products in the db
        self.db_conn.execute("INSERT INTO Products (PNum, GName, PDesc, SQaunt) " +
                             "VALUES ('" +
                             self.pn_entry_value.get() + "', '" +
                             self.gn_entry_value.get() + "', '" +
                             self.pd_entry_value.get() + "', '" +
                             self.sq_entry_value.get() + "')")

        # Clear the entry boxes
        self.pn_entry.delete(0, "end")
        self.gn_entry.delete(0, "end")
        self.pd_entry.delete(0, "end")
        self.sq_entry.delete(0, "end")

        # Update list box with product list
        self.update_listbox()

    def update_listbox(self):

        # Delete items in the list box
        self.list_box.delete(0, END)

        # Get products from the db
        try:
            result = self.theCursor.execute("SELECT ID, PNum, GName, PDesc, SQuant FROM Products")

            # You receive a list of lists that hold the result
            for row in result:

                prod_id = row[0]
                prod_pnum = row[1]
                prod_gname = row[2]
                prod_pdesc = row[3]
                prod_squant = row[4]

                # Put the product in the list box
                self.list_box.insert(prod_id,
                                     prod_pnum + " " +
                                     prod_gname + " " +
                                     prod_pdesc + " " +
                                     prod_squant)

        except sqlite3.OperationalError:
            print("The Table Doesn't Exist")

        except:
            print("1: Couldn't Retrieve Data From Database")


    # Load listbox selected product into entries
    def load_product(self, event=None):

        # Get index selected which is the product id
        lb_widget = event.widget
        index = str(lb_widget.curselection()[0] + 1)

        # Store the current product index
        self.curr_product = index

        # Retrieve product list from the db
        try:
            result = self.theCursor.execute("SELECT ID, PNum, GName, PDesc, SQuant FROM Products WHERE ID=" + index)

            # You receive a list of lists that hold the result
            for row in result:
                prod_id = row[0]
                prod_pnum = row[1]
                prod_gname = row[2]
                prod_pdesc = row[3]
                prod_squant = row[4]

                # Set values in the entries
                self.pn_entry_value.set(prod_pnum)
                self.gn_entry_value.set(prod_gname)
                self.pd_entry_value.set(prod_pdesc)
                self.sq_entry_value.set(prod_squant)

        except sqlite3.OperationalError:
            print("The Table Doesn't Exist")

        except:
            print("2 : Couldn't Retrieve Data From Database")

    # Update product info
    def update_product(self, event=None):

        # Update student records with change made in entry
        try:
            self.db_conn.execute("UPDATE Products SET PNum='" +
                                self.pn_entry_value.get() +
                                "', GName='" +
                                self.gn_entry_value.get() +
                                "', PDesc='" +
                                self.pd_entry_value.get() +
                                "', SQuant='" +
                                self.sq_entry_value.get() +
                                "' WHERE ID=" +
                                self.curr_product)

            self.db_conn.commit()

        except sqlite3.OperationalError:
            print("Database couldn't be Updated")

        # Clear the entry boxes
        self.pn_entry.delete(0, "end")
        self.gn_entry.delete(0, "end")
        self.pd_entry.delete(0, "end")
        self.sq_entry.delete(0, "end")


        # Update list box with product list
        self.update_listbox()

    def __init__(self, root):

        root.title("SuperMarket Inventory Application")
        root.geometry("270x340")

        # ----- 1st Row -----
        pn_label = Label(root, text="Product Number")
        pn_label.grid(row=0, column=0, padx=10, pady=10, sticky=W)

        # Will hold the changing value stored first name
        self.pn_entry_value = StringVar(root, value="")
        self.pn_entry = ttk.Entry(root,
                                  textvariable=self.pn_entry_value)
        self.pn_entry.grid(row=0, column=1, padx=10, pady=10, sticky=W)

        # ----- 2nd Row -----
        gn_label = Label(root, text="General Name")
        gn_label.grid(row=1, column=0, padx=10, pady=10, sticky=W)

        # Will hold the changing value stored last name
        self.gn_entry_value = StringVar(root, value="")
        self.gn_entry = ttk.Entry(root,
                                  textvariable=self.gn_entry_value)
        self.gn_entry.grid(row=1, column=1, padx=10, pady=10, sticky=W)

        # ----- 2nd Row -----
        pd_label = Label(root, text="Product Description")
        pd_label.grid(row=2, column=0, padx=10, pady=10, sticky=W)

        # Will hold the changing value stored first name
        self.pd_entry_value = StringVar(root, value="")
        self.pd_entry = ttk.Entry(root,
                                  textvariable=self.pd_entry_value)
        self.pd_entry.grid(row=2, column=1, padx=10, pady=10, sticky=W)

        # ----- 3rd Row -----
        sq_label = Label(root, text="Stock Quantity")
        sq_label.grid(row=3, column=0, padx=10, pady=10, sticky=W)

        # Will hold the changing value stored last name
        self.sq_entry_value = StringVar(root, value="")
        self.sq_entry = ttk.Entry(root,
                                  textvariable=self.sq_entry_value)
        self.sq_entry.grid(row=3, column=1, padx=10, pady=10, sticky=W)

        # ----- 4th Row -----
        self.submit_button = ttk.Button(root,
                            text="Submit",
                            command=lambda: self.prod_submit())
        self.submit_button.grid(row=4, column=0,
                                padx=10, pady=10, sticky=W)

        self.update_button = ttk.Button(root,
                            text="Update",
                            command=lambda: self.update_product())
        self.update_button.grid(row=4, column=1,
                                padx=10, pady=10)

        # ----- 5th Row -----

        scrollbar = Scrollbar(root)

        self.list_box = Listbox(root)

        self.list_box.bind('<<ListboxSelect>>', self.load_product)

        self.list_box.insert(1, "Products Here")

        self.list_box.grid(row=5, column=0, columnspan=4, padx=10, pady=10, sticky=W+E)

        # Call for database to be created
        self.setup_db()

        # Update list box with student list
        self.update_listbox()

# Get the root window object
root = Tk()

# Create the calculator
prodDB = ProductDB(root)

# Run the app until exited
root.mainloop()
