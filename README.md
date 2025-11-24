import tkinter as tk
from tkinter import messagebox, PhotoImage
import speech_recognition as sr
import pyttsx3
import random
import json
import os



# --------------------- Setup Text-to-Speech ---------------------
engine = pyttsx3.init()
engine.setProperty("rate", 150)

def speak(text):
    engine.say(text)
    engine.runAndWait()

# --------------------- Tips with Images ---------------------
tips = [
    {"text": "50-30-20 Rule for Smart Saving", "img": "https://i.imgur.com/3XJmYfZ.png"},
    {"text": "Track every expense to avoid overspending", "img": "https://i.imgur.com/7VBBsNk.png"},
]

# --------------------- Voice Input ---------------------
def get_voice_input():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        speak("Listening...")
        audio = recognizer.listen(source)
    try:
        voice_text = recognizer.recognize_google(audio)
        user_input.set(voice_text)
        handle_send()
    except Exception:
        speak("Sorry, I couldn't understand.")
        messagebox.showerror("Voice Error", "Couldn't recognize your voice.")

# --------------------- Goal Storage ---------------------
goal_file = "goals.json"
if os.path.exists(goal_file):
    with open(goal_file, "r") as f:
        goals = json.load(f)
else:
    goals = []

# --------------------- Bot Response ---------------------
def handle_send():
    text = user_input.get().strip()
    if not text:
        return

    chat_log.insert(tk.END, "You: " + text + "\n")
    response = "Type 'Set Goal', 'View Goals', or 'Tip'"

    if "set goal" in text.lower():
        response = "Enter goal name & amount (e.g. Phone 30000)"
    elif text.lower().count(" ") == 1 and text.split()[1].isdigit():
        name, amount = text.split()
        goals.append({"name": name, "target": int(amount), "saved": 0})
        with open(goal_file, "w") as f:
            json.dump(goals, f)
        response = f"Goal '{name}' added! 🎯"
    elif "view goals" in text.lower():
        if not goals:
            response = "No goals yet!"
        else:
            response = "Here are your goals:\n"
            for g in goals:
                status = f"{g['name']}: ₹{g['saved']} / ₹{g['target']}\n"
                response += status
                speak(status)
    elif "tip" in text.lower():
        tip = random.choice(tips)
        response = tip["text"]
        display_image(tip["img"])
    else:
        response = "Say or type: Set Goal, View Goals, or Tip"

    chat_log.insert(tk.END, "Bot: " + response + "\n\n")
    speak(response)
    user_input.set("")

# --------------------- Show Image ---------------------
def display_image(img_url):
    import requests
    from PIL import Image, ImageTk
    from io import BytesIO
    try:
        img_data = requests.get(img_url).content
        img = Image.open(BytesIO(img_data))
        img = img.resize((200, 120))
        img = ImageTk.PhotoImage(img)
        image_label.config(image=img)
        image_label.image = img
    except Exception:
        image_label.config(text="Failed to load image")

# --------------------- GUI Setup ---------------------
app = tk.Tk()
app.title("Smart Budget Chatbot 💰")
app.geometry("500x600")

chat_log = tk.Text(app, bg="#F0F0F0", wrap=tk.WORD)
chat_log.pack(padx=10, pady=10, fill=tk.BOTH, expand=True)

image_label = tk.Label(app)
image_label.pack()

user_input = tk.StringVar()

input_frame = tk.Frame(app)
input_frame.pack(pady=5, fill=tk.X, padx=10)

entry = tk.Entry(input_frame, textvariable=user_input, width=40, font=("Arial", 12))
entry.pack(side=tk.LEFT, padx=(0, 5), expand=True, fill=tk.X)

send_btn = tk.Button(input_frame, text="Send", command=handle_send, bg="#4CAF50", fg="white")
send_btn.pack(side=tk.LEFT)

mic_btn = tk.Button(input_frame, text="🎤", command=get_voice_input, bg="#2196F3", fg="white")
mic_btn.pack(side=tk.LEFT, padx=(5, 0))

chat_log.insert(tk.END, "Bot: Hi! I'm your Smart Budget Goal Tracker 💰\n\n")
speak("Hi! I'm your Smart Budget Goal Tracker")

app.mainloop()
