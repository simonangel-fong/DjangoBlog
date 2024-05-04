# DjangoBlog: Enable Environment variables

[Back](../../README.md)

- [DjangoBlog: Enable Environment variables](#djangoblog-enable-environment-variables)
  - [Introduction](#introduction)
  - [Install Dependency](#install-dependency)
  - [Update settings.py](#update-settingspy)
  - [Create .env for Environment Variables](#create-env-for-environment-variables)
  - [Update Deployment Configuration](#update-deployment-configuration)
  - [Summary](#summary)

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
- Commit and push project code to GitHub.

---

## Update Deployment Configuration

Update the deployment configuration after enabling environment variables in the project code.

- Clone project code

```sh
# Remove existing project directory and its contents
rm -rf /home/ubuntu/DjangoBlog-Deploy-EC2

# Clone the GitHub repository into the specified directory
git clone https://github.com/simonangel-fong/DjangoBlog-Deploy-EC2.git /home/ubuntu/DjangoBlog-Deploy-EC2
```

- Create .env file

Create a .env file in the project directory with the necessary configuration for the application

```conf
SECRET_KEY=django_secret
ALLOWED_HOSTS=127.0.0.1,localhost,ec2_public_ip,dns_name
DEBUG=True
DATABASE_URL=mysql://<user_name>:<pwd>@localhost:3306/<db_name>
```

> Notes:
>
> - `SECRET_KEY` should match the key used when starting the Django project.
> - `ALLOWED_HOSTS` contains a comma-separated list of allowed hostnames or IP addresses. Do not include spaces in the list as they could lead to connection issues.
>   At this stage, set `DEBUG=True` for development or troubleshooting, but it should be set to `False` in production.
>   `DATABASE_URL` maps to a database connection string. Use secure credentials and connection settings.

- Test the Application:

Run the server to verify the application functions correctly with the new .env configuration:

```sh
python3 /home/ubuntu/DjangoBlog-Deploy-EC2/DjangoBlog/manage.py runserver 0.0.0.0:8000
```

Access the DjangoBlog application in a browser and test creating, updating, publishing, and deleting blog posts to validate the database connection.

- Reload `Nginx`, `Gunicorn`, and `Supervisor`:

Perform the following commands to apply the changes and ensure services are running with the updated configuration:

```sh
# Test the Nginx configuration for syntax errors and other issues
sudo nginx -t

# Reload the Nginx service to apply changes in configuration without stopping the service
sudo systemctl reload nginx

# Restart the Gunicorn service to apply changes
sudo systemctl restart gunicorn

# Instruct Supervisor to reread its configuration files to recognize any new or updated configurations
sudo supervisorctl reread

# Reload Supervisor to apply the changes from the reread configuration files and restart any affected processes
sudo supervisorctl reload
```

- Test Application:

After reloading services, test the application using its public IP address or domain name.

---

## Summary

This document outlines the process of configuring the DjangoBlog project for deployment by enabling environment variables using the `django-environ` package. It involves setting up environment variables in a `.env` file and excluding the file from version control.

Deployment requires updating project code by cloning the GitHub repository and creating a .env file with necessary configurations such as `SECRET_KEY`, `ALLOWED_HOSTS`, and `DATABASE_URL`. Testing the application is done using `manage.py runserver` to ensure functionality. The document also details reloading `Nginx`, `Gunicorn`, and `Supervisor` to apply changes and advises testing the application using its public IP or domain name to confirm successful deployment.

---

[TOP](#djangoblog-enable-environment-variables)
