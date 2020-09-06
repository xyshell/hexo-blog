---
title: email-note
date: 2020-09-05 20:37:37
tags:
toc: true
---

# Reference

- [Send email in Python](https://julien.danjou.info/sending-emails-in-python-tutorial-code-examples/)
- [简单三步，用 Python 发邮件](https://zhuanlan.zhihu.com/p/24180606)

# Send email from gmail:

Turn on [Less secure app access](https://myaccount.google.com/lesssecureapps)

```python
import smtplib
from email.mime.text import MIMEText
import ssl

port = 465
password = input("your password")
context = ssl.create_default_context()

msg = MIMEText("The body of the email is here")
msg['Subject'] = "An Email Alert"
msg['From'] = "my@gmail.com"
msg['To'] = "other@xxx.xxx"

with smtplib.SMTP_SSL("smtp.gmail.com", port, context=context) as server:
    server.login("my@gmail.com", password)
    server.send_message(msg)
```
