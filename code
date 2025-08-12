import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
from datetime import datetime
import random
import csv


conn = sqlite3.connect("library.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS books(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT,
    author TEXT,
    available INTEGER DEFAULT 1
)
""")
cursor.execute("""
CREATE TABLE IF NOT EXISTS members(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT
)
""")
cursor.execute("""
CREATE TABLE IF NOT EXISTS transactions(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    book_id INTEGER,
    member_id INTEGER,
    issue_date TEXT,
    return_date TEXT,
    fine INTEGER DEFAULT 0
)
""")
conn.commit()

def add_book():
    title = book_title.get()
    author = book_author.get()
    if title and author:
        cursor.execute("INSERT INTO books (title, author) VALUES (?, ?)", (title, author))
        conn.commit()
        messagebox.showinfo("Success", "Book added.")
        refresh_books()
    else:
        messagebox.showerror("Error", "Please fill both fields.")

def delete_book():
    selected = book_tree.focus()
    if selected:
        book_id = book_tree.item(selected)['values'][0]
        cursor.execute("DELETE FROM books WHERE id = ?", (book_id,))
        conn.commit()
        messagebox.showinfo("Deleted", "Book removed.")
        refresh_books()

def refresh_books():
    book_tree.delete(*book_tree.get_children())
    for row in cursor.execute("SELECT * FROM books"):
        book_tree.insert("", "end", values=row)
    refresh_member_list()
    refresh_transactions()

def search_books():
    keyword = search_entry.get().lower()
    book_tree.delete(*book_tree.get_children())
    cursor.execute("SELECT * FROM books")
    for row in cursor.fetchall():
        if keyword in row[1].lower():
            book_tree.insert("", "end", values=row)

def add_member():
    name = member_name.get()
    if name:
        cursor.execute("INSERT INTO members (name) VALUES (?)", (name,))
        conn.commit()
        member_name.delete(0, 'end')
        messagebox.showinfo("Added", "Member registered.")
        refresh_member_list()
    else:
        messagebox.showerror("Error", "Enter a member name.")

def refresh_member_list():
    member_tree.delete(*member_tree.get_children())
    cursor.execute("SELECT id, name FROM members")
    for row in cursor.fetchall():
        member_tree.insert("", "end", values=row)
    cursor.execute("SELECT id, name FROM members")
    members = [f"{m[0]} - {m[1]}" for m in cursor.fetchall()]
    member_list["values"] = members

def select_member_from_list():
    selected = member_list.get()
    if selected:
        member_id = selected.split(" - ")[0]
        issue_member_id.delete(0, tk.END)
        issue_member_id.insert(0, member_id)

def issue_book():
    bid = issue_book_id.get()
    mid = issue_member_id.get()
    cursor.execute("SELECT available FROM books WHERE id = ?", (bid,))
    result = cursor.fetchone()
    if result and result[0] == 1:
        today = datetime.now().strftime("%Y-%m-%d")
        cursor.execute("INSERT INTO transactions (book_id, member_id, issue_date) VALUES (?, ?, ?)",
                       (bid, mid, today))
        cursor.execute("UPDATE books SET available = 0 WHERE id = ?", (bid,))
        conn.commit()
        messagebox.showinfo("Issued", "Book issued.")
        refresh_books()
    else:
        messagebox.showerror("Error", "Book not available or invalid ID.")

def assign_random_ids():
    cursor.execute("SELECT id FROM books WHERE available = 1")
    books = cursor.fetchall()
    cursor.execute("SELECT id FROM members")
    members = cursor.fetchall()
    if books and members:
        bid = random.choice(books)[0]
        mid = random.choice(members)[0]
        issue_book_id.delete(0, tk.END)
        issue_book_id.insert(0, str(bid))
        issue_member_id.delete(0, tk.END)
        issue_member_id.insert(0, str(mid))
        messagebox.showinfo("Assigned", f"Book ID: {bid}, Member ID: {mid}")
    else:
        messagebox.showwarning("Warning", "No available books or members.")

def assign_book_to_member():
    bid = assign_book_id.get()
    if not bid.isdigit():
        messagebox.showerror("Error", "Enter a valid Book ID.")
        return
    cursor.execute("SELECT available FROM books WHERE id = ?", (bid,))
    result = cursor.fetchone()
    if not result:
        messagebox.showerror("Error", "Book ID not found.")
        return
    if result[0] == 0:
        messagebox.showwarning("Unavailable", "Book is already issued.")
        return
    cursor.execute("SELECT id FROM members")
    members = cursor.fetchall()
    if not members:
        messagebox.showerror("Error", "No members found.")
        return
    mid = random.choice(members)[0]
    today = datetime.now().strftime("%Y-%m-%d")
    cursor.execute("INSERT INTO transactions (book_id, member_id, issue_date) VALUES (?, ?, ?)",
                   (bid, mid, today))
    cursor.execute("UPDATE books SET available = 0 WHERE id = ?", (bid,))
    conn.commit()
    messagebox.showinfo("Assigned", f"Book ID {bid} assigned to Member ID {mid}.")
    refresh_books()

def return_book():
    tid = return_transaction_id.get()
    cursor.execute("SELECT issue_date, book_id FROM transactions WHERE id = ? AND return_date IS NULL", (tid,))
    result = cursor.fetchone()
    if result:
        issue_date = datetime.strptime(result[0], "%Y-%m-%d")
        return_date = datetime.now()
        days = (return_date - issue_date).days
        fine = max(0, (days - 7) * 5)
        cursor.execute("UPDATE transactions SET return_date = ?, fine = ? WHERE id = ?",
                       (return_date.strftime("%Y-%m-%d"), fine, tid))
        cursor.execute("UPDATE books SET available = 1 WHERE id = ?", (result[1],))
        conn.commit()
        messagebox.showinfo("Returned", f"Book returned. Fine: ₹{fine}")
        refresh_books()
    else:
        messagebox.showerror("Error", "Invalid or already returned transaction.")

def export_transactions():
    cursor.execute("SELECT * FROM transactions")
    with open("transactions_export.csv", "w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(["ID", "Book ID", "Member ID", "Issue Date", "Return Date", "Fine"])
        for row in cursor.fetchall():
            writer.writerow(row)
    messagebox.showinfo("Exported", "Transactions saved as 'transactions_export.csv'.")

def refresh_transactions():
    trans_tree.delete(*trans_tree.get_children())
    cursor.execute("SELECT * FROM transactions")
    for row in cursor.fetchall():
        trans_tree.insert("", "end", values=row)


root = tk.Tk()
root.title("Library Book Manager")

tabs = ttk.Notebook(root)
tab1 = ttk.Frame(tabs)
tab2 = ttk.Frame(tabs)
tab3 = ttk.Frame(tabs)
tab4 = ttk.Frame(tabs)
tabs.add(tab1, text="Books")
tabs.add(tab2, text="Issue")
tabs.add(tab3, text="Return")
tabs.add(tab4, text="Members")
tabs.pack(expand=1, fill="both")


tk.Label(tab1, text="Title").grid(row=0, column=0)
book_title = tk.Entry(tab1)
book_title.grid(row=0, column=1)
tk.Label(tab1, text="Author").grid(row=1, column=0)
book_author = tk.Entry(tab1)
book_author.grid(row=1, column=1)
tk.Button(tab1, text="Add Book", command=add_book).grid(row=2, column=0, columnspan=2)

book_tree = ttk.Treeview(tab1, columns=("ID", "Title", "Author", "Available"), show="headings")
for col in book_tree["columns"]:
    book_tree.heading(col, text=col)
book_tree.grid(row=3, column=0, columnspan=2)
tk.Button(tab1, text="Delete Selected", command=delete_book).grid(row=4, column=0, columnspan=2)

tk.Label(tab1, text="Search Title").grid(row=5, column=0)
search_entry = tk.Entry(tab1)
search_entry.grid(row=5, column=1)
tk.Button(tab1, text="Search", command=search_books).grid(row=6, column=0, columnspan=2)


tk.Label(tab2, text="Book ID").grid(row=0, column=0)
issue_book_id = tk.Entry(tab2)
issue_book_id.grid(row=0, column=1)

tk.Label(tab2, text="Member ID").grid(row=1, column=0)
issue_member_id = tk.Entry(tab2)
issue_member_id.grid(row=1, column=1)

tk.Button(tab2, text="Issue Book", command=issue_book).grid(row=2, column=0, columnspan=2)
tk.Button(tab2, text="Assign Random IDs", command=assign_random_ids).grid(row=3, column=0, columnspan=2)

tk.Label(tab2, text="Assign Book ID").grid(row=4, column=0)
assign_book_id = tk.Entry(tab2)
assign_book_id.grid(row=4, column=1)
tk.Button(tab2, text="Assign Random Member", command=assign_book_to_member).grid(row=5, column=0, columnspan=2)

trans_tree = ttk.Treeview(tab2, columns=("ID", "Book ID", "Member ID", "Issue Date", "Return Date", "Fine"), show="headings")
for col in trans_tree["columns"]:
    trans_tree.heading(col, text=col)
trans_tree.grid(row=6, column=0, columnspan=2)

tk.Button(tab2, text="Export Transactions", command=export_transactions).grid(row=7, column=0, columnspan=2)

tk.Label(tab3, text="Transaction ID").grid(row=0, column=0)
return_transaction_id = tk.Entry(tab3)
return_transaction_id.grid(row=0, column=1)
tk.Button(tab3, text="Return Book", command=return_book).grid(row=1, column=0, columnspan=2)

tk.Label(tab4, text="Member Name").grid(row=0, column=0)
member_name = tk.Entry(tab4)
member_name.grid(row=0, column=1)
tk.Button(tab4, text="Add Member", command=add_member).grid(row=1, column=0, columnspan=2)

member_tree = ttk.Treeview(tab4, columns=("ID", "Name"), show="headings")
for col in member_tree["columns"]:
    member_tree.heading(col, text=col)
member_tree.grid(row=2, column=0, columnspan=2)

tk.Label(tab4, text="Select Member").grid(row=3, column=0)
member_list = ttk.Combobox(tab4)
member_list.grid(row=3, column=1)
tk.Button(tab4, text="Select for Issue", command=select_member_from_list).grid(row=4, column=0, columnspan=2)


refresh_books()


root.mainloop()
