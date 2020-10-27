# Description

A Hello World app shipped to AWS using Docker from dev to production environnements.

## TODO 

- Enable a Docker Registry
- Configure environment settings for stagging and production
- Ensure to enforce the twelve-factor methodology

## Preparing the environment
In order to run the code we will set up a virtual environment using [venv](https://docs.python.org/fr/3/library/venv.html).

We create our environment with:

```bash
> python3 -m venv venv
```

and we enable it with:

```bash
> source .venv/bin/activate
```

Note: `venv` folder is ignored by Git (see `.gitignore`)

## Requirements

We install the project requirements from the `requirements.txt`:

```bash
> pip install -r requirements.txt
```


# How to make it: 101 notes

## 1. Initialization of repository
### 1.1 Creation on Github
The repository is created on Github with a `gitignore` set for `python` and a CC 0 license.

### 1.2 Pull the repository
Then the repository was cloned in a local folder:

```bash
> git clone https://github.com/bbouille/hello-world-django-aws.git
```

## 2. Set the environnement

We create our environment with:

```bash
> python3 -m venv venv
```

and we enable it with:

```bash
> source .venv/bin/activate
```

## 3. Initialization of Django app
### 3.1 Create the project

This project named `hello` is created with: 

```bash
> django-admin startproject hello
```

Project root structure: 

```bash
(venv) ~/Code/Django/hello-world-django-aws $ tree
.
└── hello
    ├── hello
    │   ├── __init__.py
    │   ├── asgi.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── manage.py
```

### 3.2 Create the app
Inside the root `hello` folder, the app named `world` is created with: 

```bash
> cd hello
> python manage.py startapp world
```

Project and application root structure: 

```bash
(venv) ~/Code/Django/DEMO/hello $ tree
.
├── hello
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
└── world
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    └── views.py
```


### 3.3 Set a single view

#### 3.3.1 Create the index view

Add this code to `world/views.py` file:

```bash
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.

def index(request):
    return HttpResponse("Hello, world. You're at the site.")
```

#### 3.3.2 Set the URLs

Create the `world/urls.py` file with this code to set the index view:

```bash
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

Update the `hello/urls.py` file with this code:

```bash
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('', include('world.urls')),
    path('admin/', admin.site.urls),
]
```

#### 3.3.3 Check the view

Start the web server to check the view:

```bash
> python manage.py runserver
```

The page should be displayed on `http://localhost:8000/` with the `Hello, world` text.


## 4. Enable Postgres database

### 4.1 Install package

Install `postgres` package :

```bash
> pip install pip install psycopg2
```

### 4.2 Update requirements

```bash
> pip freeze > requirements.txt
```

This should contains now:

```bash
asgiref==3.2.10
Django==3.1.1
psycopg2==2.8.6
pytz==2020.1
sqlparse==0.3.1
```

### 4.3 Update project settings

Open `hello/settings.py` and replace the `sqlite3` settings in the `DATABASES` section with the following:

```bash
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}	
```

Note: these default values are set in Docker configuration introduced in the next section


## 5 Initialization of Docker

### 5.1 Create Dockerfile

Create a new file named `Dockerfile` in the root directory with this content:

```bash
# The first instruction is what image we want to base our container on
# We Use an official Python runtime as a parent image
FROM python:3.8

# The enviroment variable ensures that the python output is set straight
# to the terminal with out buffering it first
ENV PYTHONUNBUFFERED 1

# create root directory for our project in the container
RUN mkdir /web_app

# Set the working directory to /web_app
WORKDIR /web_app

# Copy the current directory contents into the container at /music_service
ADD . /web_app/

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt
```

### 5.2 Initiate Docker composer

To associate two containers, one for the web app and the other for the database, create a news file named `docker-compose.yml` with this content: 

```bash
version: "3.8"

services:
  db:
    image: postgres
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
  web:
    build: .
    command: bash -c "python manage.py makemigrations && python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/web_app
    ports:
      - "8000:8000"
    depends_on:
      - db
```

### 5.3 App execution

We run `web` and `db` containers with:

```bash
> docker-compose up
```

The page should be still displayed on `http://localhost:8000/` with the `Hello, world` text ! No modification from web browser.
