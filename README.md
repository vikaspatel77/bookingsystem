# bookingsystem
from tkinter import *
from tkinter import messagebox
import os
from tkcalendar import DateEntry
from datetime import date
import sqlite3
import random
import string


def register():
    global register_screen
    register_screen = Toplevel(main_screen)
    register_screen.title("Register")
    register_screen.geometry("800x550")
    register_screen.configure(bg='#333333')
    global email
    global password
    global name
    global phoneno
    global email_entry
    global password_entry
    global name_entry
    global phoneno_entry
    name = StringVar()
    password = StringVar()
    email = StringVar()
    phoneno = StringVar()

    Label(register_screen, text="Please enter details below",
          bg='#333333', fg="#ff3399", font=("calibri", 20)).pack()
    name_lable = Label(register_screen, text="Name ",
                       bg='#333333', fg="#ff3399", font=10)
    name_lable.pack()
    name_entry = Entry(register_screen, textvariable=name)
    name_entry.pack()
    email_lable = Label(register_screen, text="EMAIL ",
                        bg='#333333', fg="#ff3399", font=10)
    email_lable.pack()
    email_entry = Entry(register_screen, textvariable=email)
    email_entry.pack()
    phoneno_lable = Label(register_screen, text="Phonenumber  ",
                          bg='#333333', fg="#ff3399", font=10)
    phoneno_lable.pack()
    phoneno_entry = Entry(register_screen, textvariable=phoneno)
    phoneno_entry.pack()
    password_lable = Label(register_screen, text="Password  ",
                           bg='#333333', fg="#ff3399", font=10)
    password_lable.pack()
    password_entry = Entry(register_screen, textvariable=password, show='*')
    password_entry.pack()
    Button(register_screen, text="Register", width=10, height=1, bg="#333333",
           fg="#ff3399", font=10, command=register_user).pack(pady=10)


def login():
    global login_screen
    login_screen = Toplevel(main_screen)
    login_screen.title("Login")
    login_screen.geometry("500x400")
    login_screen.configure(bg='#333333')
    Label(login_screen, text="Please enter details below to login",
          bg="#333333", fg="#ff3399", font=25).pack()
    global username_verify
    global password_verify

    username_verify = StringVar()
    password_verify = StringVar()
    global username_login_entry
    global password_login_entry
    Label(login_screen, text="Username  ", bg="#333333", fg="#ff3399").pack()
    username_login_entry = Entry(login_screen, textvariable=username_verify)
    username_login_entry.pack()

    Label(login_screen, text="Password  ", bg="#333333", fg="#ff3399").pack()
    password_login_entry = Entry(
        login_screen, textvariable=password_verify, show='*')
    password_login_entry.pack()
    Button(login_screen, text="Login", width=10, height=1, bg="#333333",
           fg="#ff3399", command=login_verify).pack(pady=10)


def register_user():
    email_info = email.get()
    password_info = password.get()
    name_info = name.get()
    phoneno_info = phoneno.get()

    file = open(email_info, "w")
    file.write(name_info + "\n")
    file.write(email_info + "\n")
    file.write(password_info + "\n")
    file.write(phoneno_info)
    file.close()

    email_entry.delete(0, END)
    password_entry.delete(0, END)
    name_entry.delete(0, END)
    phoneno_entry.delete(0, END)

    Label(register_screen, text="Registration Success",
          bg='#333333', fg="#ff3399", font=("calibri", 25)).pack()


def login_verify():
    username1 = username_verify.get()
    password1 = password_verify.get()

    username_login_entry.delete(0, END)
    password_login_entry.delete(0, END)

    list_of_files = os.listdir()
    if username1 in list_of_files:
        file1 = open(username1, "r")
        verify = file1.read().splitlines()
        if password1 in verify:
            login_sucess()

        else:
            password_not_recognised()

    else:
        user_not_found()


def login_sucess():
    global login_success_screen
    top = Tk()
    top.geometry('550x300')
    top.configure(bg='#333333')

    top.title('Ticket Management:')

    # connecting to database
    conn = sqlite3.connect('ticket_booking_database.db')
    cursor = conn.cursor()
    cursor.execute(
        "CREATE TABLE IF NOT EXISTS ticket (name TEXT, ticket_id TEXT PRIMARY KEY, ticket_date TEXT, ticket_validity TEXT)")

    # fetching database
    cursor.execute('SELECT * FROM ticket')
    tickets = cursor.fetchall()

    # getting all tickets id
    global tickets_id
    tickets_id = []
    for i in tickets:
        tickets_id.append(i[1])

    conn.commit()

    Label(top, text='Ticket Management System', bg='#333333', fg="#ff3399", font=(
        'Arial', 18)).grid(row=0, column=0, columnspan=2, padx=80, pady=20)

    # defining functions
    # this function will be used to show messages and errors
    def show_message(title, message):
        messagebox.showerror(title, message)

    # this function will be used to generate random ticket id
    def get_random_string():
        letters = string.ascii_lowercase
        return ''.join(random.choice(letters) for i in range(8))

    # this function will be used to book a new ticket
    def Book():

        top1 = Tk()
        top1.geometry('350x300')
        top1.configure(bg='#333333')
        top1.title('Book')

        name = StringVar(top1)
        ticket_id = StringVar(top1)
        ticket_date = StringVar(top1)
        ticket_date.set(date.today())
        ticket_validity = StringVar(top1)

        # this while loop will create a new unique ticket id using get_random_string function
        while True:
            global tickets_id
            t_id = get_random_string()
            if t_id not in tickets_id:
                ticket_id.set(get_random_string())
                break
            continue

        def BookNow():
            if len(name.get()) < 2 or len(ticket_date.get()) < 7 or len(ticket_validity.get()) < 7:
                show_message('Error', 'Enter valid details')
                return
            try:
                conn = sqlite3.connect("ticket_booking_database.db")
                cursor = conn.cursor()
                cursor.execute("INSERT INTO ticket (name, ticket_id, ticket_date, ticket_validity) VALUES (?, ?, ?, ?)", (str(
                    name.get()), str(ticket_id.get()), str(ticket_date.get()), str(ticket_validity.get())))
                conn.commit()
                show_message('Successful', 'Your booking is successful, your ticket id is {}'.format(
                    ticket_id.get()))
                top1.destroy()
            except sqlite3.Error as e:
                show_message('Error', e)
            finally:
                conn.close()

        Label(top1, text='Enter details', bg='#333333', fg="#ff3399", font=(
            'Arial', 14)).grid(row=0, column=0, padx=10, pady=10, columnspan=2)

        Label(top1, text='Name', bg='#333333', fg="#ff3399", font=('Arial', 12)).grid(
            row=1, column=0, padx=10, pady=10, sticky='w')
        Entry(top1, textvariable=name).grid(row=1, column=1)

        Label(top1, text='Ticket Id', bg='#333333', fg="#ff3399", font=(
            'Arial', 12)).grid(row=2, column=0, padx=10, pady=10, sticky='w')
        Entry(top1, textvariable=ticket_id,
              state='disabled').grid(row=2, column=1)

        Label(top1, text='Booking Date', bg='#333333', fg="#ff3399", font=(
            'Arial', 12)).grid(row=3, column=0, padx=10, sticky='w', pady=10)
        DateEntry(top1, selectmode='day',
                  textvariable=ticket_date).grid(row=3, column=1)

        Label(top1, text='Validity', bg='#333333', fg="#ff3399", font=(
            'Arial', 12)).grid(row=4, column=0, padx=10, pady=10, sticky='w')
        DateEntry(top1, selectmode='day',
                  textvariable=ticket_validity).grid(row=4, column=1)

        Button(top1, text='Confirm', bg='#333333', fg='#ff3399', font=(
            'Arial', 17), width=9, command=lambda: BookNow()).grid(row=5, column=1, pady=10)

    # this function will be used to display all booked tickets
    def ViewHistory():

        top2 = Tk()
        top2.geometry('737x300')
        top2.configure(bg='#333333')
        top2.title('View Ticket Booking History')

        Label(top2, text='Customer Name', bg='#333333', fg="#ff3399", font=(
            'Arial', 12), borderwidth=1, relief="solid", width=20).grid(row=0, column=0, pady=10)
        Label(top2, text='Ticket ID', bg='#333333', fg="#ff3399", font=('Arial', 12),
              borderwidth=1, relief="solid", width=20).grid(row=0, column=1, pady=10)
        Label(top2, text='Date of Booking', bg='#333333', fg="#ff3399", font=(
            'Arial', 12), borderwidth=1, relief="solid", width=20).grid(row=0, column=2, pady=10)
        Label(top2, text='Ticket Validity(Date)', bg='#333333', fg="#ff3399", font=(
            'Arial', 12), borderwidth=1, relief="solid", width=20).grid(row=0, column=3, pady=10)

        conn = sqlite3.connect('ticket_booking_database.db')
        cursor = conn.cursor()

        cursor.execute('SELECT * FROM ticket')
        tickets = cursor.fetchall()
        for i in range(len(tickets)):
            Label(top2, text=tickets[i][0], borderwidth=1, bg='#333333',
                  fg="#ff3399", relief="solid", width=20).grid(row=i+1, column=0)
            Label(top2, text=tickets[i][1], borderwidth=1, bg='#333333', fg="#ff3399",
                  relief="solid", width=20).grid(row=i+1, padx=10, pady=2, column=1)
            Label(top2, text=tickets[i][2], borderwidth=1, bg='#333333', fg="#ff3399",
                  relief="solid", width=20).grid(row=i+1, padx=10, pady=2, column=2)
            Label(top2, text=tickets[i][3], borderwidth=1, bg='#333333', fg="#ff3399",
                  relief="solid", width=20).grid(row=i+1, padx=10, pady=2, column=3)
        top2.mainloop()
        conn.close()

    # this function will be used to delete a ticket
    def DeleteBooking():

        top3 = Tk()
        top3.geometry('785x300')
        top3.configure(bg='#333333')
        top3.title('View Ticket Booking History')

        Label(top3, text='Customer Name', bg='#333333', fg="#ff3399", font=(
            'Arial', 12), borderwidth=1, relief="solid", width=20).grid(row=0, column=0, pady=10)
        Label(top3, text='Ticket ID', bg='#333333', fg="#ff3399", font=('Arial', 12),
              borderwidth=1, relief="solid", width=20).grid(row=0, column=1, pady=10)
        Label(top3, text='Date of Booking', bg='#333333', fg="#ff3399", font=(
            'Arial', 12), borderwidth=1, relief="solid", width=20).grid(row=0, column=2, pady=10)
        Label(top3, text='Ticket Validity(Date)', bg='#333333', fg="#ff3399", font=(
            'Arial', 12), borderwidth=1, relief="solid", width=20).grid(row=0, column=3, pady=10)

        def delete_rows(ticket_id):
            try:
                conn = sqlite3.connect("ticket_booking_database.db")
                cursor = conn.cursor()
                cursor.execute(
                    "DELETE FROM ticket WHERE ticket_id = ?", (ticket_id,))
                conn.commit()
                show_message('Success', 'Ticket deleted',
                             bg='#333333', fg="#ff3399")
                conn.close()
            except sqlite3.Error as e:
                show_message('Sqlite error', e)
            finally:
                conn.close()
        conn = sqlite3.connect('ticket_booking_database.db')
        cursor = conn.cursor()

        cursor.execute('SELECT * FROM ticket')
        tickets = cursor.fetchall()
        for i in range(len(tickets)):
            Label(top3, text=tickets[i][0], borderwidth=1, bg='#333333',
                  fg="#ff3399", relief="solid", width=20).grid(row=i+1,  column=0)
            Label(top3, text=tickets[i][1], borderwidth=1, bg='#333333', fg="#ff3399",
                  relief="solid", width=20).grid(row=i+1,  padx=10, pady=2, column=1)
            Label(top3, text=tickets[i][2], borderwidth=1, bg='#333333', fg="#ff3399",
                  relief="solid", width=20).grid(row=i+1,  padx=10, pady=2, column=2)
            Label(top3, text=tickets[i][3], borderwidth=1, bg='#333333', fg="#ff3399",
                  relief="solid", width=20).grid(row=i+1,  padx=10, pady=2, column=3)
            Button(top3, text='Delete', bg='#333333', fg="#ff3399",
                   command=lambda current_id=tickets[i][1]: delete_rows(current_id)).grid(row=i+1, column=4)
        top3.mainloop()

        conn.close()

    # defining buttons
    Button(top, text='Book Ticket', bg='#333333', fg="#ff3399", font=('Arial', 14),
           command=lambda: Book(), width=12, height=2,).grid(row=1, column=0, padx=80, pady=20)

    Button(top, text='View History', bg='#333333', fg="#ff3399", font=('Arial', 14),
           command=lambda: ViewHistory(), width=12, height=2).grid(row=1, column=1, pady=20)

    Button(top, text='Delete Booking', bg='#333333', fg="#ff3399", font=('Arial', 14),
           command=lambda: DeleteBooking(), width=12, height=2).grid(row=2, column=0, pady=30)

    Button(top, text='Quit', bg='#333333', fg="#ff3399", font=('Arial', 14),
           command=lambda: top.destroy(), width=12, height=2).grid(row=2, column=1, pady=30)

    # mainloop
    top.mainloop()


def main_screen():
    global main_screen
    main_screen = Tk()
    main_screen.geometry("1466x800")
    main_screen.title("Account Login")
    main_screen.configure(bg='#333333')
    # frame=Frame(main_screen,bg='#333333')
    Label(text="LOGIN/REGISTER", font=("Arial", 40),
          bg='#333333', fg="#ff3399", pady=20).pack()
    # Label(text="").pack()
    Button(text="Login", height="1", width="20", bg='#333333',
           fg="#ff3399", font=("Arial", 17), command=login).pack()
    Label(text="", bg='#333333').pack()
    Button(text="Register", height="1", width="20", bg='#333333',
           fg="#ff3399", font=("Arial", 17), command=register).pack()
    Label(text="", bg='#333333').pack()

    main_screen.mainloop()


main_screen()
