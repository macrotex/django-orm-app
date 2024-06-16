# Making a stand-alone Django ORM project

## Overview 

This is a description of how to make a Django project whose sole
purpose is to be used by _other_ Django projects as the ORM layer.
This is called a "reusable app" in the Django documentation.

Why would you want to do this?

The main use case is when you have multiple Django projects that need
to use the _same_ database models (or even the same database
back-end). Rather than duplicating the code in each project it is
better to use a single Django "app" that is references by all the
projects.

Assumptions:

1. All of our Python code will be under the namespace `acme`. For
example, we might have payroll code under `acme.payroll`,
server-management code under `acme.servers`, etc.

2. Our stand-alone (resuable app) will be in the Python package
`acme.ourdb`.

3. You already have installed the Django package version 5.x or later.


## A. Create database Django project

Note. We follow the usual Django tutorial with only a few departures.

1. Create a new Django project called `mysite`:
   ```
   $ django-admin startproject mysite
   ```

2. Change into the newly-created project directory.  From now on all
file paths will be relative to the `mysite` directory just created.
   ```
   $ cd mysite
   ```

4. Make the app `acme/ourdb`. Also create the file `acme/__init__.py` so
testing works properly.
   ```
   $ mkdir -p acme/ourdb
   $ python manage.py startapp ourdb ./acme/ourdb
   $ touch acme/__init__.py
   ```


5. Fix the "name" of the app. Edit `./acme/ourdp/apps.py` and
change the value of `name` to `acme.ourdb`.
   ```
   # acme/ourdb/apps.py
   class OurdbConfig(AppConfig):
       default_auto_field = 'django.db.models.BigAutoField'
       name = 'acme.ourdb'  # CHANGE THIS LINE
   ```

6. Add the `acme.ourdb` app to the list `INSTALLED_APPS` in
`mysite/settings.py`. You need to do this in order for the Django app
to know about the new app.
   ```
   INSTALLED_APPS = [
       'acme.ourdb',
       'django.contrib.admin',
       'django.contrib.auth',
       ...
   ]
   ```

7. Create a model in `acme.ourdb`.
   ```
   # acme/ourdb/models.py
   from django.db import models

   class Question(models.Model):
       question_text = models.CharField(max_length=200)
       pub_date = models.DateTimeField("date published")


   class Choice(models.Model):
       question = models.ForeignKey(Question, on_delete=models.CASCADE)
       choice_text = models.CharField(max_length=200)
       votes = models.IntegerField(default=0)
   ```    

8. Make the initial migration file.
   ```
   $ python manage.py makemigrations
   Migrations for 'ourdb':
     acme/ourdb/migrations/0001_initial.py
       - Create model Question
       - Create model Choice
   ```

9. Add a test. Edit the file `acme/ourdb/tests.py` and put in this content.
   ```
   # acme/ourdb/tests.py
   from django.utils import timezone
   from django.test import TestCase

   from acme.ourdb.models import Question

   class QuestionTestCase(TestCase):
       def setUp(self):
           Question.objects.create(question_text="How are you?", pub_date=timezone.now())
           Question.objects.create(question_text="What is two plus two?", pub_date=timezone.now())

       def test_question_exists(self):
           """"""
           questions = Question.objects.all()
           self.assertEqual(len(questions), 2)
   ```

10. Run the tests. 
   ```
   $ python manage.py test
   Creating test database for alias 'default'...
   System check identified no issues (0 silenced).
   .
   ----------------------------------------------------------------------
   Ran 1 test in 0.001s

   OK
   Destroying test database for alias 'default'...
   ```

## B. Package and install database Django project

You can use your favorite Python packaging process but for this
tutorial we use `setuptools`.

1. While in the top-level directory `mysite` (the one with `manage.py`)
create a minimal `pyproject.toml` file.
   ```
   [build-system]
   requires = ["setuptools>=61.0", "wheel"]
   build-backend = "setuptools.build_meta"

   [tool.setuptools]
   include-package-data = true

   [tool.setuptools.packages.find]
   where = ["."]
   include = ["acme*"]

   [project]
   name = "acme_ourdb"
   version = "1.0.0"
   authors = [
     { name="Jane Roe", email="jroe@example.com" },
   ]
   description = "Acme Database Models"
   readme = "README.md"
   requires-python = ">=3.9"
   ```

2. Create the Python package.
   ```
   $ pip install build twine
   $ python -m build
   ```

3. The above build step creates a Python "wheel" file that is left in
the `dist/` directory.
   ```
   $ ls dist/
   acme_ourdb-1.0.0-py3-none-any.whl  acme_ourdb-1.0.0.tar.gz
   ```

4. Install the package.
   ```
   $ pip install dist/acme_ourdb-1.0.0-py3-none-any.whl
   ```

## C. Use `acme.ourdb` in another Django package

1. Change directory to a location where you will put a new Django project.
Our new Django project will be called `myweb`.
   ```
   $ cd /some/appropriate/path
   $ django-admin startproject myweb
   ```

2. Chanage into the new directory `myweb/`.
   ```
   $ cd myweb
   ```

3. Add the `acme.ourdb` app to `myweb`'s installed apps by
editing `myweb/settings.py` and adding it to the list
   ```
   INSTALLED_APPS = [
       'acme.ourdb',
       'django.contrib.admin',
       'django.contrib.auth',
       ...
   ]
   ```

4. Run the migrations. This will create the `acme.ourdb` tables in the
database.
   ```
   $ python manage.py migrate
   Operations to perform:
     Apply all migrations: admin, auth, contenttypes, ourdb, sessions
   Running migrations:
     Applying contenttypes.0001_initial... OK
     Applying auth.0001_initial... OK
     Applying admin.0001_initial... OK
     ...
     Applying ourdb.0001_initial... OK     <-- the acme.ourdb migration
     Applying sessions.0001_initial... OK
   ```

5. Create a superuser.
   ```
   $ python manage.py createsuperuser
   Username (leave blank to use 'myuser'): admin
   Email address: admin@example.com
   Password:
   Password (again):
   Superuser created successfully.
   ```

6. Run the local web server to ensure everything is working.
   ```
   $ python manage.py runserver
   ```

7. Open `http://127.0.0.1:8000/admin` and login using the
superuser name and password.

8. Create an app within `myweb`. We will create the `main` app.
   ```
   $ python manage.py startapp main
   ```

9. Add this new app to `myweb`'s list of installed apps.
   ```
   INSTALLED_APPS = [
       'main',
       'acme.ourdb',
       'django.contrib.admin',
       'django.contrib.auth',
       ...
   ]
   ```

10. Register the `acme.ourdb` models in the `main` admin
by editing `main/adminp.py`.
   ```
   # main/adminp.py
   from django.contrib import admin
   from acme.ourdb.models import Question, Choice

   admin.site.register(Question)
   admin.site.register(Choice)
   ```

11. Open the admin site again and verify that you can edit the
Question and Choice models.

12. After you make changes to the database scheme in `acme.ourdb` you
need to package, install, and then run the migration in each other
project that uses `acme.ourdb`.
