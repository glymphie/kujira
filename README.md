<div align="center">

# 🐋 Kujira 🐋

</div>

<div align="center">

  <img src="https://i.imgur.com/RKz5MlN.png" alt="Kujira" align="center" width=400>

</div>

<div align="center">

### A "simple" and secure Docker-compose setup with a TLS proxy, supporting Django development and production environments.

</div>

## Setup 📝

#### Clone and cd to repo

```
git clone https://github.com/glymphie/kujira.git && cd kujira
```

#### Create a new Django project in the src folder:

```
django-admin startproject yourprojectname src
```

#### Add / change this in your `src/yourprojectname/settings.py`

```diff
+import os

+SECRET_KEY = os.environ['DJANGO_SECRET_KEY']
-SECRET_KEY = 'django-insecure-k-emo17%oafwv8l1lh6aene@zxgu+r0+hbpzid91f24d#yk4&f'

+ALLOWED_HOSTS = ['*']
-ALLOWED_HOSTS = []

-DATABASES = {
-    'default': {
-        'ENGINE': 'django.db.backends.sqlite3',
-        'NAME': BASE_DIR / 'db.sqlite3',
-    }
-}

+DATABASES = {
+    'default': {
+       'ENGINE': 'django.db.backends.postgresql_psycopg2',
+       'NAME': os.environ['POSTGRES_DB'],
+       'USER': os.environ['POSTGRES_USER'],
+       'PASSWORD': os.environ['POSTGRES_PASSWORD'],
+       'HOST': os.environ['POSTGRES_HOST'],
+       'PORT': os.environ['POSTGRES_PORT'],
+   },
+}

+CSRF_TRUSTED_ORIGINS = ['https://0.0.0.0']
```

#### Environment file:

Create the environment file `.env` from the provided template:

```
cp env.template .env
```

And change the values inside `.env` as you see fit:

```
DJANGO_SECRET_KEY='django_web_key'
POSTGRES_DB='main'
POSTGRES_USER='username'
POSTGRES_PASSWORD='password'
```

#### Generate Certificate and Private Key:
- **Certbot:** Follow instructions from https://certbot.eff.org/
- **Locally:**

```
openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out certificate.pem -keyout key.pem
```

Create DH Parameters:

```
openssl genpkey -genparam -algorithm DH -out dhparam.pem -pkeyopt dh_paramgen_prime_len:4096
```

Move these 3 new files into the `nginx` folder. Alternatively, create symlinks.

```
mv *.pem nginx/
```


## How to run 🚀

```
docker-compose up
```

## Common errors 🤔
#### Djangos Admin CSRF token

```
Forbidden (403)

CSRF verification failed. Request aborted.
```

see:
- https://docs.djangoproject.com/en/4.1/ref/csrf/
- https://docs.djangoproject.com/en/4.0/ref/settings/#csrf-trusted-origins

---

#### CSP errors

> The default django page is all weird.

The Content Security Policy (CSP) does not allow inline CSS by default.

And the default django page is inherently insecure because it uses inline CSS.
(Who would have thought?)

**I advice to not dig much deeper into this issue with the default page.**

Just implement it into your own site and don't try to get the default django page to load its css.

see:
- https://django-csp.readthedocs.io/en/latest/index.html
- https://www.stackhawk.com/blog/django-content-security-policy-guide-what-it-is-and-how-to-enable-it/
- https://realpython.com/django-nginx-gunicorn/#adding-a-content-security-policy-csp-header
- https://content-security-policy.com/examples/allow-inline-style/


