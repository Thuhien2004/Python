# Python
Bài tập 1 - Lập trình python - Nguyễn Thị Thu Hiền - K2254801016015
<3 Dưới đây là kết quả bài tập

![Screenshot 2025-05-07 152706](https://github.com/user-attachments/assets/c921d994-bb12-42d2-8556-d0327ca385d1)

![Screenshot 2025-05-07 152836](https://github.com/user-attachments/assets/a5fbbdd8-13a1-46bf-99aa-1a4fdaa28e31)



CODE
---------------------------------------------------------------------------------------------------------------------------------
import smtplib
import imaplib
import email
from email.mime.text import MIMEText

# -----------------------------------------------
# Bước 1: Đọc thông tin tài khoản từ tệp văn bản
# -----------------------------------------------

# Đọc thông tin tài khoản từ file email_credentials.txt
with open('email_credentials.txt', 'r') as file:
    account_info = file.readlines()
    email_account = account_info[0].strip()  # Đọc email từ dòng đầu tiên
    app_password = account_info[1].strip()  # Đọc App Password từ dòng thứ hai

# -----------------------------------------------
# Bước 2: Đọc nội dung email, thông tin người nhận và tiêu chí lọc email từ các tệp văn bản
# -----------------------------------------------

# Đọc nội dung email gửi đi từ file email_content.txt
with open('email_content.txt', 'r', encoding='utf-8') as file:
    email_content = file.read() # Đọc toàn bộ nội dung email

# Đọc địa chỉ người nhận từ file recipient.txt
with open('recipient.txt', 'r') as file:
    recipient_email = file.read().strip()  # Đọc địa chỉ email người nhận

# Đọc tiêu chí lọc email (địa chỉ người gửi) từ file email_filter.txt
with open('email_filter.txt', 'r') as file:
    filter_sender = file.read().strip()  # Đọc địa chỉ email người gửi cần lọc

# -----------------------------------------------
# Bước 3: Gửi email qua Gmail
# -----------------------------------------------

# Tạo nội dung email
msg = MIMEText(email_content)
msg['Subject'] = 'Email Tự Động'
msg['From'] = email_account
msg['To'] = recipient_email

# Kết nối và gửi email qua máy chủ SMTP của Gmail
try:
    with smtplib.SMTP('smtp.gmail.com', 587) as server:
        server.starttls()  # Kích hoạt mã hóa TLS
        server.login(email_account, app_password)  # Đăng nhập bằng email và App Password
        server.send_message(msg)  # Gửi email
        print(f'Email đã được gửi đến {recipient_email}')
except Exception as e:
    print(f"Không thể gửi email: {e}")

# -----------------------------------------------
# Bước 4: Nhận email từ hộp thư đến
# -----------------------------------------------

try:
    # Kết nối đến máy chủ IMAP của Gmail
    with imaplib.IMAP4_SSL('imap.gmail.com') as server:
        server.login(email_account, app_password)  # Đăng nhập bằng email và App Password
        server.select('INBOX')  # Chọn hộp thư đến

        # Tìm kiếm email từ người gửi cụ thể (theo địa chỉ email trong file email_filter.txt)
        _, data = server.search(None, f'FROM "{filter_sender}"')

        # Duyệt qua các email tìm được và hiển thị thông tin
        for num in data[0].split():
            _, msg_data = server.fetch(num, '(RFC822)')
            email_msg = email.message_from_bytes(msg_data[0][1])
            subject = email_msg['subject']
            print(f"Tiêu đề email nhận được: {subject}")

            # Lấy nội dung email (chỉ văn bản thuần túy)
            if email_msg.is_multipart():
                for part in email_msg.walk():
                    if part.get_content_type() == 'text/plain':
                        print(f"Nội dung: {part.get_payload(decode=True).decode()}")
            else:
                print(f"Nội dung: {email_msg.get_payload(decode=True).decode()}")
            break  # Chỉ in email đầu tiên để đơn giản
except Exception as e:
    print(f"Không thể nhận email: {e}")
