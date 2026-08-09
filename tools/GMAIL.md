# Gmail SMTP

The `.env` file provides `GMAIL_SMTP_USER` and `GMAIL_SMTP_APP_PASSWORD`. Load them into the environment without printing them:

```bash
set -a
source .env
set +a
```

Send an email with Python's standard library:

```bash
EMAIL_TO="recipient@example.com" SUBJECT="Subject" BODY="Message text" python3 - <<'PY'
import os
import smtplib
from email.message import EmailMessage

message = EmailMessage()
message["From"] = os.environ["GMAIL_SMTP_USER"]
message["To"] = os.environ["EMAIL_TO"]
message["Subject"] = os.environ["SUBJECT"]
message.set_content(os.environ["BODY"])

with smtplib.SMTP("smtp.gmail.com", 587, timeout=30) as smtp:
    smtp.starttls()
    smtp.login(
        os.environ["GMAIL_SMTP_USER"],
        os.environ["GMAIL_SMTP_APP_PASSWORD"],
    )
    smtp.send_message(message)
PY
```

Use the Gmail address as the sender. Keep `.env` private and never log or commit the app password.
