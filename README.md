# Nodejs-project
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Email Reminder System</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f3f4f6;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

    .container {
      background: white;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      width: 400px;
    }

    h2 {
      text-align: center;
      color: #333;
    }

    label {
      font-weight: bold;
      display: block;
      margin-top: 10px;
    }

    input, textarea, button {
      width: 100%;
      padding: 10px;
      margin-top: 5px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }

    button {
      background-color: #007bff;
      color: white;
      border: none;
      cursor: pointer;
      margin-top: 15px;
    }

    button:hover {
      background-color: #0056b3;
    }

    .success {
      color: green;
      margin-top: 10px;
      text-align: center;
    }
  </style>
</head>
<body>

  <div class="container">
    <h2>Email Reminder System</h2>
    <form id="reminderForm">
      <label for="email">Recipient Email:</label>
      <input type="email" id="email" placeholder="Enter email" required>

      <label for="subject">Subject:</label>
      <input type="text" id="subject" placeholder="Enter subject" required>

      <label for="message">Message:</label>
      <textarea id="message" rows="5" placeholder="Enter reminder details" required></textarea>

      <label for="date">Reminder Date:</label>
      <input type="datetime-local" id="date" required>

      <button type="submit">Set Reminder</button>
      <p class="success" id="successMessage"></p>
    </form>
  </div>

  <script>
    document.getElementById("reminderForm").addEventListener("submit", function(e){
      e.preventDefault();

      const email = document.getElementById("email").value;
      const subject = document.getElementById("subject").value;
      const message = document.getElementById("message").value;
      const date = document.getElementById("date").value;

      if (!email || !subject || !message || !date) {
        alert("Please fill in all fields!");
        return;
      }

      // Simulate saving reminder
      document.getElementById("successMessage").innerText = 
        "✅ Reminder set successfully for " + new Date(date).toLocaleString();

      // In a real system, you’d send this data to your server:
      // fetch('/send-reminder', { method: 'POST', body: JSON.stringify({ email, subject, message, date }) })
    });
  </script>

from flask import Flask, render_template, request, redirect, flash
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime
import threading
import time

app = Flask(__name__)
app.secret_key = "supersecretkey"  # Needed for flash messages


# Configure your email account here
SENDER_EMAIL = "youremail@gmail.com"
SENDER_PASSWORD = "yourpassword"  # App password if using Gmail


def send_email(to_email, subject, message):
    """Function to send an email using SMTP."""
    try:
        msg = MIMEMultipart()
        msg["From"] = SENDER_EMAIL
        msg["To"] = to_email
        msg["Subject"] = subject
        msg.attach(MIMEText(message, "plain"))

        with smtplib.SMTP("smtp.gmail.com", 587) as server:
            server.starttls()
            server.login(SENDER_EMAIL, SENDER_PASSWORD)
            server.send_message(msg)

        print(f"✅ Email sent to {to_email}")
    except Exception as e:
        print(f"❌ Error sending email: {e}")


def schedule_email(to_email, subject, message, send_time):
    """Thread that waits until send_time and then sends the email."""
    def task():
        now = datetime.now()
        delay = (send_time - now).total_seconds()
        if delay > 0:
            time.sleep(delay)
        send_email(to_email, subject, message)

    threading.Thread(target=task).start()


@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        email = request.form["email"]
        subject = request.form["subject"]
        message = request.form["message"]
        date_str = request.form["date"]

        # Convert to datetime object
        send_time = datetime.fromisoformat(date_str)

        # Schedule the email
        schedule_email(email, subject, message, send_time)
        flash(f"✅ Reminder set for {send_time.strftime('%Y-%m-%d %H:%M:%S')}")
        return redirect("/")

    return render_template("index.html")


if __name__ == "__main__":
    app.run(debug=True)

</body>
</html>
