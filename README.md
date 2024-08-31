# python_remainder_app
import pickle
import datetime
import time
from playsound import playsound
import tkinter as tk
from tkinter import messagebox
import threading

# Reminder class to handle each reminder
class Reminder:
    def __init__(self, message, time, interval=None):
        self.message = message
        self.time = time
        self.interval = interval  # Interval in minutes for repeated reminders

    def __repr__(self):
        return f"Reminder('{self.message}', '{self.time}', '{self.interval}')"

# Function to save reminders to a file
def save_reminders(reminders, filename='reminders.pkl'):
    with open(filename, 'wb') as f:
        pickle.dump(reminders, f)

# Function to load reminders from a file
def load_reminders(filename='reminders.pkl'):
    try:
        with open(filename, 'rb') as f:
            return pickle.load(f)
    except FileNotFoundError:
        return []

# Function to check and trigger reminders
def check_reminders(reminders):
    while True:
        current_time = datetime.datetime.now()
        for reminder in reminders:
            if reminder.time <= current_time:
                print(f"Reminder: {reminder.message}")
                playsound('alarm.wav')
                if reminder.interval:
                    reminder.time += datetime.timedelta(minutes=reminder.interval)
                else:
                    reminders.remove(reminder)
        save_reminders(reminders)
        time.sleep(30)

# GUI class for the Reminder App
class ReminderApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Reminder App")
        
        self.reminders = load_reminders()
        
        self.label = tk.Label(root, text="Enter your reminder details:")
        self.label.pack()

        self.message_label = tk.Label(root, text="Message:")
        self.message_label.pack()
        self.message_entry = tk.Entry(root)
        self.message_entry.pack()

        self.time_label = tk.Label(root, text="Time (YYYY-MM-DD HH:MM):")
        self.time_label.pack()
        self.time_entry = tk.Entry(root)
        self.time_entry.pack()

        self.interval_label = tk.Label(root, text="Interval (minutes, optional):")
        self.interval_label.pack()
        self.interval_entry = tk.Entry(root)
        self.interval_entry.pack()

        self.add_button = tk.Button(root, text="Add Reminder", command=self.add_reminder)
        self.add_button.pack()

        self.start_button = tk.Button(root, text="Start Checking", command=self.start_checking)
        self.start_button.pack()

    def add_reminder(self):
        message = self.message_entry.get()
        time_str = self.time_entry.get()
        interval = self.interval_entry.get()

        try:
            reminder_time = datetime.datetime.strptime(time_str, "%Y-%m-%d %H:%M")
            if interval:
                interval = int(interval)
            else:
                interval = None
            new_reminder = Reminder(message, reminder_time, interval)
            self.reminders.append(new_reminder)
            save_reminders(self.reminders)
            messagebox.showinfo("Reminder App", "Reminder added successfully!")
        except ValueError:
            messagebox.showerror("Reminder App", "Invalid time format. Please use YYYY-MM-DD HH:MM.")

    def start_checking(self):
        threading.Thread(target=check_reminders, args=(self.reminders,)).start()

# Main function to run the GUI application
if __name__ == "__main__":
    root = tk.Tk()
    app = ReminderApp(root)
    root.mainloop()
