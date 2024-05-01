# DjangoBlog Project 04: Enable Environment variables

[Back](../../README.md)

- [DjangoBlog Project 04: Enable Environment variables](#djangoblog-project-04-enable-environment-variables)
  - [Introduction](#introduction)
  - [Install Dependency](#install-dependency)
  - [Update settings.py](#update-settingspy)
  - [Create .env for Environment Variables](#create-env-for-environment-variables)

---

## Introduction

In developing the DjangoBlog project, careful configuration is necessary for effective deployment. This document outlines the process of updating the `settings.py` file to use environment variables through the `environ` library, creating a `.env` file to securely store these variables, and adding the `.env` file to the `.gitignore` file to exclude it from version control.

---

## Install Dependency

To manage environment variables effectively in the Django project, install the `django-environ` package that allows to work with environment variables and configurations easily.

- ref: https://django-environ.readthedocs.io/en/latest/quickstart.html

```sh
# install dependency
pip install django-environ
```

---

## Update settings.py

To use environment variables, update the `settings.py` file as follows:

```py
# Import the environ library:
import environ

# Initialize environment variables and read from the .env file:
env = environ.Env(DEBUG=(bool, False))
environ.Env.read_env()

# Set the values using environment variables:
SECRET_KEY = env('SECRET_KEY')
DEBUG = env('DEBUG')

ALLOWED_HOSTS = env.list('ALLOWED_HOSTS') # list multiple hosts

# Database
# Configure the database settings using the DATABASE_URL environment variable, defaulting to SQLite for development:
DATABASES = {
    'default': env.db_url(
        # refer to .env DATABASE_URL
        'DATABASE_URL',
        # by default, use sqlite3 for dev
        default=f'sqlite:///{BASE_DIR}/db.sqlite3'
    )
}
```

---

## Create .env for Environment Variables

- create a `.env` file in the directory of the project, which will store environment variables.

```conf
SECRET_KEY=secret_key
ALLOWED_HOSTS=127.0.0.1, localhost, some_host
DEBUG=True
#DATABASE_URL=db_url.
```

> Refer to the official documentation for more details on database URL configuration. https://django-environ.readthedocs.io/en/latest/api.html

- Add `.env` file to `.gitignore`

---

[TOP](#djangoblog-project-04-enable-environment-variables)
