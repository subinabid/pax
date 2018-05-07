# My first django app

### Install Python
I am using a windows machine for development. Python for windows can be downloaded from https://www.python.org/downloads/

The version is 3.6.5

### Pip
pip comes bundled with this version of Python

### Installing django
`pip install django` installs djnago

### Initiate Project
`django-admin startproject projectname` starts the project.
This will create the required directories and the `manage.py` file

### Initiate App - polls in this case


### App -  add view
In `polls/view.py` , add the code

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index")
```

### App - URLs
Create a a file `polls/urls.py` and add the code

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

Similarly, in `project/urls.py` add the code

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
### Database
By default, django uses SQLite, which takes care of itself.

I made an attempt to host my app on Redhat's Openshift, which uses a django + PostgreSQL combination. I will figure it out later.

### Time zones
Default time zone is American. I haven't figured out how to change it to Indian time zone. Read some docs which says its as simple as going to `project/settings.py` and changing `TIME_ZONE = 'UTC'` to `TIME_ZONE = 'IN'`. But didn't work for me.

### Models
Models defines the database layouts

in `polls/models.py`, add the following code:

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

### Add our app to the django app list
Add our app (polls) to `INSTALLED_APPS` section of `project/settings.py`. The documentation says I should add `'polls.apps.PollsConfig',`. Simply adding `'polls',` would work fine as well.

Run `python manage.py makemigrations polls` to tell pyhton about the new Models

Run `python manage.py migrate` to apply changes to the database

### A bit about Git

I have been using github for desktop for a  while, but decided to try the CLI instead. By the time you add your models, django starts generating its own files in `__pycache__` folder. I guess I need not add those files in `git add`. But every time I use `git status`, it lists a few untracked files, which is irritating. I need to figure out how to get rid of those!

### Markdown syntax formatting

while using ``` ``` to highlight code, we specify the language use to format the code. ```Python ``` does not work the same way as ```python ```. The latter is what we want. I learned is the hard way.
