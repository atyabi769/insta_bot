from instagram_private_api import Client, ClientCompatPatch
import time
import json
import os

# اطلاعات ورود به اینستاگرام
username = 'your_username'
password = 'your_password'

# ورود به حساب کاربری
api = Client(username, password)

# آی‌دی ریلز مورد نظر
reel_id = 'your_reel_id'

# متن پیام تبلیغاتی
message_text = """
سلام! 🌟
این یک پیام تبلیغاتی است.
برای مشاهده صفحه ما به لینک زیر مراجعه کنید:
https://www.instagram.com/your_page
"""

# فایل ذخیره‌سازی کاربران
sent_users_file = 'sent_users.json'

# بررسی و بارگذاری لیست کاربران از فایل
if os.path.exists(sent_users_file):
    with open(sent_users_file, 'r') as file:
        sent_users = json.load(file)
else:
    sent_users = []

# دریافت لایک‌ها و کامنت‌ها
likers = api.media_likers(reel_id)
likers_ids = [user['pk'] for user in likers['users']]
comments = api.media_comments(reel_id)
commenters_ids = [comment['user']['pk'] for comment in comments['comments']]

# ترکیب لیست لایک‌کنندگان و کامنت‌گذاران
target_users = list(set(likers_ids + commenters_ids))

# ارسال پیام به ۱۰ نفر در روز و جلوگیری از تکرار
daily_limit = 10
sent_today = 0

for user_id in target_users:
    if user_id not in sent_users:
        try:
            api.direct_message(user_id, message_text)
            sent_users.append(user_id)
            sent_today += 1
            if sent_today >= daily_limit:
                break
        except Exception as e:
            print(f"Error sending message to user {user_id}: {e}")
    else:
        continue

# ذخیره‌سازی لیست کاربران به‌روز شده در فایل
with open(sent_users_file, 'w') as file:
    json.dump(sent_users, file)

print("پیام‌ها ارسال شدند")
