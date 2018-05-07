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

```Python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index")
```

### App - URLs
Create a a file `polls/urls.py` and add the code

```Python
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

```Python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
