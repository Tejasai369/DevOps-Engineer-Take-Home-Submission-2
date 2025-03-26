#ALERTING STRATEGY & SYSTEM MONITORING SCRIPT


import time
import psutil
import requests
import smtplib
from email.mime.text import MIMEText

# Slack Webhook URL - Replace with your actual webhook
SLACK_WEBHOOK_URL = "YOUR_SLACK_WEBHOOK_URL"

# Email Configuration
SMTP_SERVER = "smtp.gmail.com"  # Use "smtp.office365.com" for Outlook
SMTP_PORT = 587
EMAIL_SENDER = "your_email@gmail.com"
EMAIL_PASSWORD = "your_email_password"  # Use an app password for security
EMAIL_RECEIVER = "recipient_email@gmail.com"

# Alert Conditions
CPU_THRESHOLD = 85  # CPU usage in %
MEMORY_THRESHOLD = 90  # Memory usage in %
DISK_THRESHOLD = 10  # Available disk space in %

DURATION = 300  # 5 minutes (300 seconds)
CHECK_INTERVAL = 10  # Check system stats every 10 seconds

# Function to send Slack alerts
def send_slack_alert(alert_message):
    payload = {"text": alert_message}
    try:
        response = requests.post(SLACK_WEBHOOK_URL, json=payload)
        response.raise_for_status()
        print("✅ Slack Alert Sent!")
    except requests.exceptions.RequestException as e:
        print(f"❌ Slack Alert Failed: {e}")

# Function to send Email alerts
def send_email_alert(subject, message):
    try:
        msg = MIMEText(message)
        msg["Subject"] = subject
        msg["From"] = EMAIL_SENDER
        msg["To"] = EMAIL_RECEIVER

        server = smtplib.SMTP(SMTP_SERVER, SMTP_PORT)
        server.starttls()
        server.login(EMAIL_SENDER, EMAIL_PASSWORD)
        server.sendmail(EMAIL_SENDER, EMAIL_RECEIVER, msg.as_string())
        server.quit()

        print("✅ Email Alert Sent!")
    except Exception as e:
        print(f"❌ Email Alert Failed: {e}")

# Function to monitor system usage
def monitor_system():
    cpu_exceeded_time = 0
    memory_exceeded_time = 0
    disk_exceeded_time = 0

    while True:
        cpu_usage = psutil.cpu_percent(interval=1)
        memory_usage = psutil.virtual_memory().percent
        disk_usage = psutil.disk_usage('/').percent

        print(f"CPU: {cpu_usage}%, Memory: {memory_usage}%, Disk: {100 - disk_usage}% free")

        # CPU Alert
        if cpu_usage > CPU_THRESHOLD:
            cpu_exceeded_time += CHECK_INTERVAL
        else:
            cpu_exceeded_time = 0

        if cpu_exceeded_time >= DURATION:
            alert_msg = f"🚨 High CPU Usage Alert! CPU at {cpu_usage}% for over 5 minutes."
            send_slack_alert(alert_msg)
            send_email_alert("High CPU Usage Alert!", alert_msg)
            cpu_exceeded_time = 0

        # Memory Alert
        if memory_usage > MEMORY_THRESHOLD:
            memory_exceeded_time += CHECK_INTERVAL
        else:
            memory_exceeded_time = 0

        if memory_exceeded_time >= DURATION:
            alert_msg = f"🚨 High Memory Usage Alert! Memory at {memory_usage}% for over 5 minutes."
            send_slack_alert(alert_msg)
            send_email_alert("High Memory Usage Alert!", alert_msg)
            memory_exceeded_time = 0

        # Disk Space Alert
        if (100 - disk_usage) < DISK_THRESHOLD:
            disk_exceeded_time += CHECK_INTERVAL
        else:
            disk_exceeded_time = 0

        if disk_exceeded_time >= DURATION:
            alert_msg = f"🚨 Low Disk Space Alert! Only {100 - disk_usage}% available for over 5 minutes."
            send_slack_alert(alert_msg)
            send_email_alert("Low Disk Space Alert!", alert_msg)
            disk_exceeded_time = 0

        time.sleep(CHECK_INTERVAL)

if __name__ == "__main__":
    monitor_system()
