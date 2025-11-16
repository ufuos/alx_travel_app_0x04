ALX Travel App 0x04 — Deployment Documentation 

Purpose: This document explains how to set up, configure, and deploy ALX Travel App 0x04 using Django 4.2, Gunicorn, Docker, and Render.com. 

Table of Contents

Requirements (requirements.txt)

Project structure

Environment & sensitive settings

Production-ready settings.py notes

Procfile

Dockerfile

Render configuration (render.yaml)

Auto-create Django superuser (idempotent)

Environment variables (Render Dashboard)

Deployment checklist

Optional enhancements & next steps

License & contact

1. Requirements — Django 4.2 Compatible

Save the following as requirements.txt at the repository root.

Django>=4.2,<5
gunicorn==20.1.0
dj-database-url==1.2.0
psycopg2-binary==2.9.10
whitenoise==6.4.0
djangorestframework==3.15.0
django-cors-headers==4.0.0
python-dotenv==1.0.0
stripe
Pillow
django-crispy-forms
cloudinary
django-celery-beat
celery[redis]
redis

Notes:

psycopg2-binary simplifies PostgreSQL usage on Render.

The list includes commonly-used packages for REST, CORS, Stripe, Cloudinary, Celery + Redis support.

Pin versions where this matters for reproducible builds.

2. Recommended Project Structure
alx_travel_app_0x04__project/
├── alx_travel_app_0x04__project/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── local_settings.py   # gitignored
├── bookings/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   ├── forms.py
│   ├── admin.py
│   └── templates/bookings/
├── core/  # or `trips`, `users`, etc.
├── templates/
├── static/
├── media/  # local dev only
├── requirements.txt
├── Dockerfile
├── Procfile
├── render.yaml
├── manage.py
└── README.md

Adjust app names to match your codebase. Keep local_settings.py out of version control.

3. Environment & Sensitive Settings (local_settings.py)

DO NOT commit local_settings.py or real secrets. Add to .gitignore.

Example local_settings.py (for local development only):

SECRET_KEY = "your-secret"
DEBUG = True
ALLOWED_HOSTS = ["localhost", "127.0.0.1"]
DATABASE_URL = "postgres://user:password@localhost:5432/alx_travel_app"
STRIPE_PUBLIC_KEY = ""
STRIPE_SECRET_KEY = ""
EMAIL_HOST = "smtp.gmail.com"
EMAIL_HOST_USER = "you@gmail.com"
EMAIL_HOST_PASSWORD = "yourpassword"
DEFAULT_FROM_EMAIL = "you@example.com"
CLOUDINARY_URL = "cloudinary://..."
CELERY_BROKER_URL = "redis://..."
CELERY_RESULT_BACKEND = "redis://..."

For production (Render), use environment variables instead.

4. Production-Ready Settings (notes)

Key items to ensure in settings.py:

Read SECRET_KEY, DEBUG, ALLOWED_HOSTS, DATABASE_URL from environment variables using dj_database_url or django-environ.

Configure DATABASES via dj_database_url.parse(os.environ['DATABASE_URL']).

Configure static files with WhiteNoise:

Add whitenoise.middleware.WhiteNoiseMiddleware near top of MIDDLEWARE.

Set STATIC_ROOT = BASE_DIR / 'staticfiles' and run collectstatic during deployment.

Add CORS_ALLOWED_ORIGINS or CORS_ALLOW_ALL_ORIGINS as appropriate and install django-cors-headers.

Secure production settings:

SESSION_COOKIE_SECURE = True

CSRF_COOKIE_SECURE = True

SECURE_BROWSER_XSS_FILTER = True

SECURE_SSL_REDIRECT = True (only if SSL is terminated)

Configure Email backend using SMTP env vars.

Configure Cloudinary for media (or S3) when CLOUDINARY_URL is present.

Example (conceptual) snippet in settings (do not copy verbatim without testing):

import dj_database_url
DATABASES = {
    'default': dj_database_url.parse(os.environ.get('DATABASE_URL'))
}


STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'


# Stripe
STRIPE_PUBLIC_KEY = os.getenv('STRIPE_PUBLIC_KEY')
STRIPE_SECRET_KEY = os.getenv('STRIPE_SECRET_KEY')
5. Procfile for Render

Create Procfile in repository root. Adjust the module path to match your project package name.

web: gunicorn alx_travel_app_0x04__project.wsgi --log-file - --workers 2 --timeout 120

Notes:

Set workers to suit your plan. 2 is reasonable for small deployments.

--log-file - streams logs to stdout for Render.

6. Dockerfile (recommended for Render with Docker environment)

Create Dockerfile at repo root.

FROM python:3.11-slim


ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1


RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    gcc \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*


WORKDIR /app


COPY requirements.txt /app/
RUN pip install --upgrade pip setuptools wheel
RUN pip install -r requirements.txt


COPY . /app/


# Collect static (best-effort; allow failure during build)
RUN python manage.py collectstatic --noinput || true


EXPOSE 8000


CMD ["sh", "-c", "python manage.py migrate --noinput && \
    gunicorn alx_travel_app_0x04__project.wsgi --bind 0.0.0.0:${PORT:-8000} --workers 2 --log-file - --timeout 120"]

Notes:

The CMD runs migrations at container start — convenient but ensure migrations are safe to auto-run.

If you prefer separate migration step or release-phase hooks, remove migrations from CMD and run them manually.

7. Render Configuration (render.yaml)

A sample render.yaml to deploy a Docker web service and optionally create a Postgres DB and worker (edit values before use).

services:
  - type: web
    name: alx-travel-app
    env: docker
    plan: free
    region: oregon
    dockerfilePath: ./Dockerfile
    branch: main
    envVars:
      - key: SECRET_KEY
        value: ""
      - key: DEBUG
        value: "False"
      - key: DATABASE_URL
        value: ""
      - key: ALLOWED_HOSTS
        value: "alx-travel-app.onrender.com"
      - key: DJANGO_SUPERUSER_USERNAME
        value: "admin"
      - key: DJANGO_SUPERUSER_EMAIL
        value: "admin@example.com"
      - key: DJANGO_SUPERUSER_PASSWORD
        value: ""
      - key: CORS_ALLOWED_ORIGINS
        value: "https://yourfrontend.com"


# Optional: add worker service for Celery
#  - type: worker
#    name: alx-travel-app-worker
#    env: docker
#    dockerfilePath: ./Dockerfile
#    startCommand: "celery -A alx_travel_app_0x04__project worker --loglevel=info"

Edit name, region, and other values to match your account.

8. Auto-Create Django Superuser on Render (idempotent)

Run the following command in the Render shell (or as a one-off deploy step). It will create the superuser only if it does not exist; if a password is provided it will update it.

python manage.py shell -c "from django.contrib.auth import get_user_model; import os; User = get_user_model(); username = os.environ.get('DJANGO_SUPERUSER_USERNAME'); email = os.environ.get('DJANGO_SUPERUSER_EMAIL'); password = os.environ.get('DJANGO_SUPERUSER_PASSWORD'); u = User.objects.filter(username=username).first();
if u is None:
    User.objects.create_superuser(username=username, email=email, password=password or 'adminpass')
else:
    if password:
        u.set_password(password); u.save()"

Notes:

Render supports running commands from the dashboard shell. You can include this in a startup script if desired.

9. Environment Variables (Render Dashboard)
Variable	Required	Description
SECRET_KEY	✔	Django production secret key
DEBUG	✔	Set to "False" in production
DATABASE_URL	✔	Render Postgres URL or other Postgres URL
ALLOWED_HOSTS	✔	Example: alx-travel-app.onrender.com
STRIPE_PUBLIC_KEY	✖	Stripe public key
STRIPE_SECRET_KEY	✖	Stripe secret key
EMAIL_HOST	✖	SMTP host (e.g., smtp.gmail.com)
EMAIL_HOST_USER	✖	SMTP username
EMAIL_HOST_PASSWORD	✖	SMTP password
DEFAULT_FROM_EMAIL	✖	From email for outgoing mail
CLOUDINARY_URL	✖	Cloudinary connection string
CELERY_BROKER_URL	✖	Redis / broker URL for Celery
CELERY_RESULT_BACKEND	✖	Result backend URL
DJANGO_SUPERUSER_USERNAME	✔	admin username for initial superuser
DJANGO_SUPERUSER_EMAIL	✔	admin email
DJANGO_SUPERUSER_PASSWORD	✔	admin password (or set manually)

Security tips:

Do not store secrets in the repository.

Use Render's encrypted environment variable store.

10. Deployment Checklist

Before pushing:




On Render:

Connect your GitHub repo to Render.

Create a Postgres database in Render (managed DB) and copy the URL to DATABASE_URL env var.

Add all required env vars in the Render Dashboard.

Deploy the service (Render will build the Docker image and start it).

Run migrations from Render shell:

python manage.py migrate
python manage.py collectstatic --noinput

Run the Superuser creation command from Shell if you didn't set DJANGO_SUPERUSER_* env vars.

Verify the app: visit the provided Render URL.

Post-deploy security:

Enable secure cookies and HTTPS redirects if applicable.

Monitor logs for errors and performance.

11. Optional Enhancements

Celery + Redis: Add a worker service in render.yaml and configure CELERY_BROKER_URL & CELERY_RESULT_BACKEND.

Cloudinary or S3: Move media files to Cloudinary (configured via CLOUDINARY_URL) or S3 for production.

DRF & JWT: If you expose APIs, consider DRF + JWT authentication for stateless clients.

CI/CD: Add a GitHub Actions workflow to run tests and static checks before deploy.

Health checks & monitoring: Add uptime checks and error reporting (Sentry).

If you'd like, I can generate any of these files for you (local_settings template, a minimal core app, DRF JWT setup, GitHub Actions workflow, or a worker render.yaml section).

12. License & Contact

License: MIT (create a LICENSE file if you want open-source permissive license)

Contact: Developer: Ufuoma Ogedegbe — https://github.com/ufuos

