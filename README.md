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


## Details

Note. We follow the usual Django tutorial with only a few departures.

1. Create a new Django project called ???:
```
$ django-admin startproject mysite
```

2. Change into the newly-created project directory.  From now on all
file paths will be relative from the `mysite` directory just created.
```
$ cd mysite
```

3. Create the `acme` parent app.
```
$ python manage.py startapp acme
```

4. Make the child app `ourdb` inside of `acme`.
```
$ mkdir acme/ourdb
$ python manage.py startapp ourdb ./acme/ourdb
```

5. Fix the "name" of the app. Edit `./acme/ourdp/apps.py` and
change the value of `name` to `acme.ourdb`.
```
# acme/ourdb/apps.py
class OurdbConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'acme.ourdb'
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
$ python manage.py tests
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```
