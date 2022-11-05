<div align="center">

# 🐋 Kujira 🐋

</div>

<div align="center">

  <img src="https://i.imgur.com/RKz5MlN.png" alt="Kujira" align="center" width=400>

</div>

<div align="center">

## A "simple" and secure Docker-compose setup with a TLS proxy, supporting Django in development and production.

</div>

## Requirements

- [docker-compose](https://github.com/docker/compose) >= 2.12.2
- [django](https://www.djangoproject.com/) >= 4.1.1
    - `python3 -m pip install --user --upgrade -r requirement.txt`


## Setup 📝

### 1. Clone and cd to the repository

```
git clone https://github.com/glymphie/kujira.git && cd kujira
```

### 2. Create a new Django project in the `src` folder:

```
django-admin startproject yourprojectname src
```

### 3. Add/change this in your `src/yourprojectname/settings.py`

```diff
from pathlib import Path
+import os

...

# SECURITY WARNING: keep the secret key used in production secret!
-SECRET_KEY = 'django-insecure-k-emo17%oafwv8l1lh6aene@zxgu+r0+hbpzid91f24d#yk4&f'
+SECRET_KEY = os.environ['DJANGO_SECRET_KEY']

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

-ALLOWED_HOSTS = []
+ALLOWED_HOSTS = ['YOUR_DOMAIN OR *']

...

# Database
# https://docs.djangoproject.com/en/4.1/ref/settings/#databases

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

...

USE_I18N = True

USE_TZ = True

+# CSRF Trusted Origins
+CSRF_TRUSTED_ORIGINS = [YOUR_TRUSTED_DOMAINS e.g. 'https://YOUR_DOMAIN/']

```

### 4. Create environment file:

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



### 5. Create DH Parameters:

This could take some time on slower PCs.

```
openssl genpkey -genparam -algorithm DH -out nginx/dhparam.pem -pkeyopt dh_paramgen_prime_len:4096
```

### 6. Generate Certificate and Private Key (A. or B.):

#### A. **Certbot:** Follow instructions from https://certbot.eff.org/

Create symlinks to the certificate and key in the `nginx` folder for `YOUR_DOMAIN`.

- `ln -s /etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem nginx/certificate.pem`
- `ln -s /etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem nginx/key.pem`

Add/change the following in the `docker-compose.yml` file

```diff
  nginx:
    image: nginx:latest
    container_name: nginx_proxy
    volumes:
      - ./nginx:/etc/nginx
+     - /etc/letsencrypt/live/YOUR_DOMAIN:/etc/letsencrypt/live/YOUR_DOMAIN:ro
+     - /etc/letsencrypt/archive/YOUR_DOMAIN:/etc/letsencrypt/archive/YOUR_DOMAIN:ro
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - web
    logging:
      driver: none
```


#### B. **Locally for testing only:**

```
openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out nginx/certificate.pem -keyout nginx/key.pem
```

### 7. DONE! 🥳


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
- https://docs.djangoproject.com/en/4.1/ref/settings/#csrf-trusted-origins

---

#### CSP errors

> The default django page is all weird.

The Content Security Policy (CSP) does not allow inline CSS by default.

And the default django page is inherently insecure because it uses inline CSS.
(Who would have thought?)

**I advice to not dig much deeper into this issue with the default page.**

Just implement your own site without inline CSS and don't try to get the default django page to load its CSS. It won't work.

see:
- https://django-csp.readthedocs.io/en/latest/index.html
- https://www.stackhawk.com/blog/django-content-security-policy-guide-what-it-is-and-how-to-enable-it/
- https://realpython.com/django-nginx-gunicorn/#adding-a-content-security-policy-csp-header
- https://content-security-policy.com/examples/allow-inline-style/


#### Bugs

For any encountered bugs or security issues, please submit an issue here on GitHub.
