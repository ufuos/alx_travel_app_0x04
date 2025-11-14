alx-travel-app-0x04 — Full Travel Booking Web App (Django 4 + Render Deployment)

A complete, production-ready travel booking application built with Django 4, PostgreSQL, Stripe payments, Celery workers (optional), Cloudinary image hosting, and fully deployable on Render.com using gunicorn.

This project allows users to browse trips, book trips, make Stripe payments, leave reviews, manage bookings, and handle email notifications — all using clean Django MVC architecture.

✨ Features

🧭 Trip Listings & Detail Pages

🧾 Full Booking System
Users can book trips, choose quantity, see price totals.

💳 Stripe Checkout Payments

⭐ Trip Reviews System

👤 User Authentication
Uses Django's built-in auth (django.contrib.auth)

🖼️ Image Uploads (Cloudinary or Local)

✉️ Email Notifications

⏳ Optional Celery + Redis Integration (for background jobs)

🌍 Ready to Deploy on Render.com

📁 Project Structure
alx_travel_app/
│
├── alx_travel_app/
│   ├── settings.py
│   ├── local_settings.py      ← NOT committed (sensitive)
│   ├── urls.py
│   ├── wsgi.py
│   ├── celery.py (optional)
│
├── bookings/
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   ├── forms.py
│   ├── admin.py
│
├── templates/
│   └── bookings/
│        ├── trip_list.html
│        ├── trip_detail.html
│        ├── book_trip.html
│        ├── payment_success.html
│        ├── payment_cancel.html
│
├── static/
├── media/ (local dev only)
│
├── requirements.txt
├── render.yaml
├── Dockerfile (optional)
├── .gitignore
└── README.md

🛠️ Installation (Local Development)
1. Clone the Repo
git clone https://github.com/<your-username>/alx-travel-app-0x04.git
cd alx-travel-app-0x04

2. Create Virtual Environment
python -m venv venv
source venv/bin/activate

3. Install Dependencies
pip install -r requirements.txt

4. Create .env for Local Dev (optional)
DEBUG=True
SECRET_KEY=your-secret-key
ALLOWED_HOSTS=localhost,127.0.0.1
DATABASE_URL=postgres://user:password@localhost:5432/alx_travel_app
STRIPE_PUBLIC_KEY=your_key
STRIPE_SECRET_KEY=your_key
EMAIL_HOST=smtp.gmail.com
EMAIL_HOST_USER=you@gmail.com
EMAIL_HOST_PASSWORD=yourpassword
DEFAULT_FROM_EMAIL=you@gmail.com
CLOUDINARY_URL=cloudinary://xxxx

5. Run Migrations
python manage.py makemigrations
python manage.py migrate

6. Create Admin User
python manage.py createsuperuser

7. Start Server
python manage.py runserver

📦 Requirements

requirements.txt:

Django>=4.2,<5
gunicorn
psycopg2-binary
django-environ
whitenoise
stripe
python-dotenv
Pillow
django-crispy-forms
cloudinary
django-celery-beat
celery[redis]
redis

🔐 Sensitive Settings — local_settings.py

Do NOT upload this file to GitHub.
Add to .gitignore.

SECRET_KEY = "your-secret"
DEBUG = True
ALLOWED_HOSTS = ["localhost", "127.0.0.1"]

DATABASE_URL = "postgres://user:password@localhost:5432/alx_travel_app"

STRIPE_PUBLIC_KEY = ""
STRIPE_SECRET_KEY = ""
EMAIL_HOST = ""
EMAIL_HOST_USER = ""
EMAIL_HOST_PASSWORD = ""
DEFAULT_FROM_EMAIL = "you@example.com"

🗄️ Database Models

Trip

Booking

Payment

Review

Profile (optional)

See bookings/models.py for full implementation.

💳 Stripe Integration

The booking flow creates a Stripe Checkout Session:

session = stripe.checkout.Session.create(
    payment_method_types=['card'],
    line_items=[{
        'price_data': {
            'currency': 'usd',
            'unit_amount': int(total * 100),
            'product_data': {'name': trip.title},
        },
        'quantity': 1,
    }],
    mode='payment',
    success_url=request.build_absolute_uri(reverse("bookings:payment_success", args=[booking.pk])),
    cancel_url=request.build_absolute_uri(reverse("bookings:payment_cancel", args=[booking.pk])),
)

📨 Email Notifications

Emails are sent using:

from django.core.mail import send_mail
send_mail(
    subject="Booking Confirmed",
    message="Your booking is confirmed.",
    from_email=settings.DEFAULT_FROM_EMAIL,
    recipient_list=[booking.user.email],
)

⏳ Background Jobs (Optional — Celery + Redis)

To enable Celery:

Add Redis URL:

CELERY_BROKER_URL=redis://…
CELERY_RESULT_BACKEND=redis://…


Worker command:

celery -A alx_travel_app worker --loglevel=info

🌐 Deployment on Render (Web + DB + Worker)
1. Create GitHub Repo

Push your project:

git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<yourname>/alx-travel-app-0x04.git
git push -u origin main

2. Go to Render → New Web Service

Settings:

Build:

pip install -r requirements.txt


Start:

gunicorn alx_travel_app.wsgi:application --bind 0.0.0.0:8000

3. Create a PostgreSQL Database in Render
4. Add Environment Variables

In Render Dashboard:

DEBUG=False
SECRET_KEY=your-secret
ALLOWED_HOSTS=your-app.onrender.com
DATABASE_URL=<Render Postgres URL>

STRIPE_PUBLIC_KEY=
STRIPE_SECRET_KEY=

EMAIL_HOST=
EMAIL_HOST_USER=
EMAIL_HOST_PASSWORD=
DEFAULT_FROM_EMAIL=

CLOUDINARY_URL=

CELERY_BROKER_URL=
CELERY_RESULT_BACKEND=

5. Run Migrations in Render Console
python manage.py migrate
python manage.py collectstatic --noinput

🧰 Optional: render.yaml

Automates deployment of:

Web service

Celery worker

Database

Included in the project.

🧪 Testing

Create tests in any app under:

app/tests/


Run tests:

python manage.py test

📌 Security Checklist (Production)

DEBUG = False

Use strong SECRET_KEY

HTTPS enforced

Cloudinary or S3 for media

Secure cookies:

SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True

📄 License

MIT License (optional — add LICENSE file if desired)

🤝 Contributions

Pull requests are welcome. For major changes, open an issue first.

📧 Contact

Developer: Ufuoma Ogedegbe
GitHub: https://github.com/ufuos