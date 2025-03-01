# Massege-box
import tkinter as tk
from tkinter import messagebox
import sqlite3
import time
import threading



# Database setup
def init_db():
    conn = sqlite3.connect("tasks.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            task TEXT NOT NULL,
            due_time INTEGER
        )
    """)
    conn.commit()
    conn.close()

# Add task
def add_task():
    task = task_entry.get()
    due_time = int(due_time_entry.get()) if due_time_entry.get().isdigit() else None
    
    if task:
        conn = sqlite3.connect("tasks.db")
        cursor = conn.cursor()
        cursor.execute("INSERT INTO tasks (task, due_time) VALUES (?, ?)", (task, due_time))
        conn.commit()
        conn.close()
        
        task_list.insert(tk.END, task)
        task_entry.delete(0, tk.END)
        due_time_entry.delete(0, tk.END)
    else:
        messagebox.showwarning("Warning", "Task cannot be empty!")

# Delete selected task
def delete_task():
    selected_task = task_list.curselection()
    if selected_task:
        task_text = task_list.get(selected_task)
        
        conn = sqlite3.connect("tasks.db")
        cursor = conn.cursor()
        cursor.execute("DELETE FROM tasks WHERE task = ?", (task_text,))
        conn.commit()
        conn.close()
        
        task_list.delete(selected_task)
    else:
        messagebox.showwarning("Warning", "No task selected!")

# Load tasks from database
def load_tasks():
    conn = sqlite3.connect("tasks.db")
    cursor = conn.cursor()
    cursor.execute("SELECT task FROM tasks")
    tasks = cursor.fetchall()
    conn.close()
    
    for task in tasks:
        task_list.insert(tk.END, task[0])

# Reminder function
def check_reminders():
    while True:
        conn = sqlite3.connect("tasks.db")
        cursor = conn.cursor()
        cursor.execute("SELECT task, due_time FROM tasks")
        tasks = cursor.fetchall()
        conn.close()
        
        current_time = int(time.time())
        for task, due_time in tasks:
            if due_time and current_time >= due_time:
                notification.notify(title="Task Reminder", message=f"{task} is due!", timeout=5)
                
                conn = sqlite3.connect("tasks.db")
                cursor = conn.cursor()
                cursor.execute("DELETE FROM tasks WHERE task = ?", (task,))
                conn.commit()
                conn.close()
        
        time.sleep(30)  # Check every 30 seconds

# UI Setup
root = tk.Tk()
root.title("To-Do List with Reminders")
root.geometry("400x400")

task_entry = tk.Entry(root, width=40)
task_entry.pack(pady=10)

due_time_entry = tk.Entry(root, width=40)
due_time_entry.pack(pady=5)
due_time_entry.insert(0, "Enter due time (UNIX timestamp)")

add_button = tk.Button(root, text="Add Task", command=add_task)
add_button.pack(pady=5)

delete_button = tk.Button(root, text="Delete Task", command=delete_task)
delete_button.pack(pady=5)

task_list = tk.Listbox(root, width=50)
task_list.pack(pady=10)

init_db()
load_tasks()

# Start reminder thread
reminder_thread = threading.Thread(target=check_reminders, daemon=True)
reminder_thread.start()

root.mainloop()
