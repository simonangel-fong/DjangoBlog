# DjangoBlog: Deploy Checklist

[Back](../../README.md)

- [DjangoBlog: Deploy Checklist](#djangoblog-deploy-checklist)
  - [Introduction](#introduction)
  - [Deployment Checklist](#deployment-checklist)
  - [Environment-Specific Settings](#environment-specific-settings)
  - [Applying Configuration for Production Environments](#applying-configuration-for-production-environments)
  - [Summary](#summary)

---

## Introduction

Properly deploying a Django project requires attention to security settings. This document provides a deployment checklist and outlines key environment-specific settings. Using `manage.py check --deploy` helps identify potential security issues.

---

## Deployment Checklist

Before deploying a Django project, it is crucial to review the settings for security, performance, and operational considerations. Django offers many built-in security features to enhance application safety.

Refer to the official document for detailed guidance: https://docs.djangoproject.com/en/5.0/howto/deployment/checklist/

- check the configuration before deploying

```sh
python ./DjangoBlog/manage.py check --deploy
```

![check](./pic/check01.png)

This command identifies potential security issues:

| Issues                | Description                                                    | Solution                            |
| --------------------- | -------------------------------------------------------------- | ----------------------------------- |
| SECRET_KEY            | Provides cryptographic signing                                 | Load from an environment variable   |
| SECURE_SSL_REDIRECT   | Redirects all non-HTTPS requests to HTTPS                      | Set True in settings.py             |
| SESSION_COOKIE_SECURE | Uses a secure cookie for the session cookie                    | Set True in settings.py             |
| CSRF_COOKIE_SECURE    | Uses a secure cookie for the CSRF cookie                       | Set True in settings.py             |
| SECURE_HSTS_SECONDS   | Applies HTTP Strict Transport Security header on all responses | Set a non-zero value in settings.py |

- Update `settings.py`

To address the security issues identified in the deployment checklist, update the settings.py file as follows:

```py
# HTTPS settings
# redirects all non-HTTPS requests to HTTPS
SECURE_SSL_REDIRECT = True
# use a secure cookie for the session cookie
SESSION_COOKIE_SECURE = True
# use a secure cookie for the CSRF cookie.
CSRF_COOKIE_SECURE = True


# HSTS(HTTP Strict Transport Security) settings
# HSTS: web browsers to only interact with the website using HTTPS, ensuring secure and encrypted connections.
# sets the HTTP Strict Transport Security header on all responses that do not already have it.
SECURE_HSTS_SECONDS = 31536000  # 1 year
# the duration of one year, for whichthe browser should remember that the website should only be accessed over HTTPS.
# enforces secure connections for an extended period, helping ensure that users can't accidentally or maliciously connect to the website using HTTP.

# adds the preload directive to the HTTP Strict Transport Security header.
# tells browsers to include domain in a preloaded list of HSTS-enabled websites.
SECURE_HSTS_PRELOAD = True

# adds the includeSubDomains directive to the HTTP Strict Transport Security header.
# nstructing browsers that not only should the main domain be accessed via HTTPS, but all of its subdomains as well.
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
```

After making these updates to the settings.py file, recheck the configuration.

![check](./pic/check02.png)

The recheck may still show issues related to `SECRET_KEY` and `DEBUG`, which should be handled via environment variables.

---

## Environment-Specific Settings

In addition to using `manage.py check --deploy` to identify potential issues before deploying, there are additional settings that need to be considered for a secure and efficient deployment.

- `ALLOWED_HOSTS`

When `DEBUG` is set to `False`, `ALLOWED_HOSTS` must be properly `configured`. Without a suitable value for this setting, Django will not work correctly, and the application could be vulnerable to CSRF attacks.

In this project, the `ALLOWED_HOSTS` setting is handled via environment variables.

- `DEBUG`

Never enable `DEBUG = True` in production environments. While it provides useful debugging features such as full tracebacks, enabling `DEBUG` in production can lead to significant security risks, including exposing sensitive information about the project.

In this project, the `DEBUG` setting is managed through environment variables.

- Static Files

Define `STATIC_ROOT` in the `settings.py` file to specify the directory where static files should be collected. Then, use the following command to collect static files into `STATIC_ROOT`:

```sh
python manage.py collectstatic -c --noinput
```

---

## Applying Configuration for Production Environments

After running the deployment checklist, the application should be configured according to production requirements. Applying the checklist settings may cause issues during development due to the enforcement of `HTTPS`. Therefore, it is recommended to apply these configurations only in production.

A practical approach is to use `GitHub` Actions to automatically apply the required settings after each code push. This ensures that the application is properly prepared for production deployment without interfering with the development process.

- Add a step in the GitHub Action yaml file.

```yml
 - name: Add HTTPS and HSTS settings to settings.py
        run: |
          # Define the path to the settings.py file
          SETTINGS_FILE=./DjangoBlog/settings.py

          # Append HTTPS and HSTS settings to settings.py
          echo "
# HTTPS settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# HSTS (HTTP Strict Transport Security) settings
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_PRELOAD = True
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
" >> $SETTINGS_FILE
```

---

## Summary

Properly deploying a Django project requires attention to security, performance, and operational settings. Use `manage.py check --deploy` to identify security issues and update `settings.py`:

- `HTTPS`: Enable `SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE`, and `CSRF_COOKIE_SECURE`.
- `HSTS`: Set `SECURE_HSTS_SECONDS`, `SECURE_HSTS_PRELOAD`, and `SECURE_HSTS_INCLUDE_SUBDOMAINS`.
- Manage `SECRET_KEY` and `DEBUG` using environment variables and configure `ALLOWED_HOSTS` to prevent `CSRF` attacks.
- For static files, define `STATIC_ROOT` and use `collectstatic`.

Use `GitHub Actions` to apply `HTTPS` and `HSTS` settings after each code push for secure and streamlined production deployment.

---

[TOP](#djangoblog-deploy-checklist)
