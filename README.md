FinalProject — это Django-приложение-витрина для онлайн-продажи цветов с корзиной, оформлением заказов, адресной книгой и уведомлениями в Telegram.
Код делится на четыре логических блока (orders, users, fileviewer, ядро flower_delivery) и использует внешние сервисы — Telegram Bot API и DaData.
Секреты читаются из config.json; база данных по-умолчанию — SQLite, развертывание рассчитано на Gunicorn + Nginx.
CI/CD, тесты и контейнеризация отсутствуют — эти задачи приоритетны для дальнейшего развития.

1. Общее описание проекта
•	Назначение — веб-магазин для оформления заказов на доставку цветов и внутренний административный интерфейс (кабинет менеджера).
•	Ключевые функциональные блоки
1.	Каталог и корзина (orders) — CRUD товаров/категорий, корзина (для гостя и авторизованного пользователя), оформление и отмена заказов (raw.githubusercontent.com).
2.	Пользователи и адреса (users) — регистрация, кастомная модель CustomUser, валидация адресов через DaData, выбор адреса по-умолчанию (raw.githubusercontent.com, raw.githubusercontent.com).
3.	Интеграции — уведомления о новых/отменённых заказах в Telegram (orders/bot_utils.py) (raw.githubusercontent.com).
4.	Просмотр файлов (fileviewer) — простейший LFI-браузер файлов сервера (для отладки; несёт риски) (raw.githubusercontent.com).

2. Структура файлов/папок (до 3-го уровня)
```   
FinalProject/
├── flower_delivery/          # Django-ядро: настройки, URL, wsgi
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── orders/                   # Каталог, корзина, Telegram-utils
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   └── bot_utils.py
├── users/                    # Пользователи, адреса, регистрация
│   ├── models.py
│   ├── views.py
│   └── forms.py
├── fileviewer/               # Браузер файлов (LFI-риск)
│   └── views.py
├── templates/                # Django templates (HTML)
├── static/                   # CSS/JS/изображения интерфейса
├── media/ products/          # Загружаемые фото букетов
├── manage.py                 # Django CLI-скрипт
├── requirements.txt          # Python-зависимости
└── README.md / LICENSE
```
```
Узел	Назначение	Особые замечания
flower_delivery/settings.py	Глобальная конфигурация проекта	Читает config.json; fallback-ключ SECRET_KEY хранится в репозитории — убрать в переменные окружения (raw.githubusercontent.com)

media/products/	Фотографии товаров	При росте объёма вынести в S3/MinIO
fileviewer/	Просмотр файлов на сервере	Нет аутентификации; можно читать произвольные файлы внутри base_dir (raw.githubusercontent.com)
```
3. Модули / пакеты (основное API)
Модуль	Роль и публичный интерфейс	Зависимости
flower_delivery.settings	Настройки Django, чтение config.json, REST-framework, логгирование	django, djangorestframework (raw.githubusercontent.com)

orders.models	Product, ProductCategory, Cart, Order, Review и связующая OrderProduct	django, users.Address, orders.bot_utils (raw.githubusercontent.com)

orders.views	40+ функций — витрина (index, product_detail), корзина (add_to_cart, cart_detail), менеджерские CRUD-handers (manage_*), checkout, отмена заказа	orders.models/forms, users.models, Telegram utils (raw.githubusercontent.com)

orders.bot_utils	notify_new_order, notify_order_cancellation → формируют текст и шлют в Telegram	requests, Telegram Bot API (raw.githubusercontent.com)

users.models	CustomUser (поле role), Address (расширенная модель адреса)	django (raw.githubusercontent.com)

users.views	Регистрация, логин с объединением гостей и пользовательской корзины, CRUD адресов, вызовы DaData	dadata, orders.models (raw.githubusercontent.com)

fileviewer.views	list_files, view_file с чёрным списком путей EXCLUDE_FILES_AND_DIRS	os, json, Django templates (raw.githubusercontent.com)

4. Ключевые переменные и конфигурация
Переменная	Откуда берётся	Значение/роль
SECRET_KEY, DEBUG, ALLOWED_HOSTS	config.json → settings.py	Секрет, режим среды, допустимые хосты (raw.githubusercontent.com)

AIORGRAM_API_TOKEN, WEBHOOK_URL	settings.py	Токен и Webhook Telegram-бота
telegram_token, chat_id	config.json → orders/bot_utils.py	Авторизация и чат для уведомлений (raw.githubusercontent.com)

dadata_token	config.json → users/views.py	Авторизация в DaData API (raw.githubusercontent.com)

EXCLUDE_FILES_AND_DIRS	config.json для fileviewer	Файлы/папки, скрываемые от просмотра (raw.githubusercontent.com)

DB	settings.DATABASES — SQLite	Просто меняется на PostgreSQL (raw.githubusercontent.com)


