---
title: "منیج کردن دیتابیس تحت وب با استفاده از adminer"
subtitle:
date: 2025-01-16T23:41:59+03:30
tags: ["Adminer", "DB"]
language: fa
cover: "/images/adminer.png"
---

# معرفی ابزار [adminer](https://www.adminer.org/)

تا چندوقت پیش هر رکوردی رو که میخواستم داخل دیتابیس روی سرور دست بزنم با psql انجام میدادم که برای کارای کوچیک اوکی بود ولی هردفعه وارد کانتینر شدنو لاگین کردن و ساپورت نکردن متن فارسی داخل ترمینال پروسه رو طولانی و رو مخ میکرد. تا چندوقت پیش که داشتم این پروژه رو نگاه میکردم [fastapi fullstack template](https://github.com/fastapi/full-stack-fastapi-template/blob/master/backend/README.md) دیدم داره از یه سرویسی به اسم adminer استفاده میکنه و از اونجا بیشتر دردسرایی که بالا گفتم واسم رفع شد

### چی هست این adminer؟

این adminer یه ابزاره که یک رابط کاربری تحت وب (webui) در اختیارتون میزاره که میتونین باهاش دیتابیستون رو مدیریت کنین. مثل pgadmin یا mysql workbech و ابزارای مشابهش.
adminer چندین تا دیتابیس از جمله mysql, postgresql, oracle, etc رو ساپورت میکنه

### مزیتش چیه؟

شما میتونین پورت دیتابیستون رو اکسپوز کنین و با [pgadmin](https://www.pgadmin.org/) به دیتابیس سرورتون وصل بشین. ولی اینکار جزو best practice ها حساب نمیشه و ریسک های امنیتی براتون بوجود میاره. در اضاش میتونین کنار کانتینر پروژتون یه کانتینر adminer بالا بیارین و اونطور adminer رو به دیتابیس پروژتون متصل کنین. اینطوری دیگه نیاز نیست پورت دیتابیستون رو اکسپوز کنین از طرفی میتونین از مزیتای امنیتی adminer مثل ریت لیمیت کردن خودکار درخواستا میشه استفاده کنین

> Security is #1 priority in development of Adminer. Adminer does not allow connecting to databases without a password and it rate-limits the connection attempts to protect against brute-force attacks. Still, consider making Adminer inaccessible to public by whitelisting IP addresses allowed to connect to it, password-protecting the access in your web server, enabling security plugins (e.g. to require an OTP) or by customizing the login method. You can also delete Adminer if not needed anymore, it is just one file which is easy to upload in the future. Adminer had some security bugs in the past so update whenever Adminer tells you there is a new version available (ask your administrator if you could not update yourself).

#### نمونه استفاده از adminer داخل یک پروژه بک اند با fastapi و postgresql:

```yaml
services:
  postgres:
    container_name: postgres
    image: postgres:16
    networks:
      - main
    # ports:
    #   - "5432:5432"
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    env_file:
      - .env

  app:
    build: .
    command:
      [
        "uvicorn",
        "config:app",
        "--host",
        "0.0.0.0",
        "--port",
        "8080"
      ]
      PYTHONUNBUFFERED: 1
    networks:
      - main
    env_file:
      - .env

  adminer:
    image: adminer:4.8.1
    container_name: adminer
    environment:
      ADMINER_DEFAULT_SERVER: postgres
      ADMINER_DESIGN: nette # Optional: Set Adminer theme
    ports:
      - "8088:8080"
    restart: unless-stopped
    networks:
      - main

networks:
  main:
    driver: bridge
```

اگه دقت کنین پورت پستگرس رو کامنت کردم و پستگرس و ادمینر رو داخل یک نتورک به اسم main گزاشتم که بتونن از طریق شبکه داخلی به هم دسترسی داشته باشن و دیگه پورت دیتابیس اکسپوز نشده. پورت دیفالت ادمینر 8080 هست که من چون سرویسم روی 8080 بود تغییر دادم به 8088 (داخل داکر پورت دادن به این صورت کار میکنه <پورت کانتینر>:<پورت هاست/سرور> اینجا سرور روی پورت 8088 درخواست رو دریافت میکنه و به پورت 8080 adminer میده و یعنی adminer همچنان از روی پورت دیفالتش درخواست رو دریافت میکنه و نیاز به دادن پارامتر بخصوصی واسه تغییر پورت بهش نیستیم)
