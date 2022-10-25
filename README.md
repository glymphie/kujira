<div align="center">

# Kujira

</div>

<div align="center">

  <img src="https://i.imgur.com/RKz5MlN.png" alt="Kujira" align="center" width=400>

</div>

<div align="center">

### A minimalistic Docker-compose setup with a TLS proxy, supporting Django development.

</div>

## How to run

https://www.linode.com/docs/guides/getting-started-with-nginx-part-4-tls-deployment-best-practices/#create-a-larger-diffie-hellman-prime

django-admin startproject projectname src

## Add this to your settings.py


## Common errors
##### Djangos Admin CSRF token

```
Forbidden (403)

CSRF verification failed. Request aborted.
```

see:
- https://docs.djangoproject.com/en/4.1/ref/csrf/
- https://docs.djangoproject.com/en/4.0/ref/settings/#csrf-trusted-origins

---

##### CSP errors

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


