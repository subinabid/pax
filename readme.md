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
    return HttpResponse("Hello, world. This is the boring page you will see when you request the polls page. we haven't reached the template part of tutorial yet!")
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

This adds a proper URL to our app. By default, django shows a beautiful index page with a rocket when the project is initiated. When we edit the `urls.py`, the beautiful index file wont be visible anymore. The system throws a 404 when you try `localhost:8000`. Hence forth, we will have to visit `localhost:8000/polls` to see the dirty view we created some time ago, or `localhost:8000/admin` to see the admin page. username and password for the admin page will be handled later

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

while using ````  ``` ``` ````  to highlight code, we specify the language to format the code. ```` ```Python ``` ```` does not work the same way as```` ```python ``` ````. The latter is what we want. I learned this the hard way.

### Creating an admin

Run `python manage.py createsuperuser` to create an admin. django will ask for a username, email id and password.

I always wondered where django stores the username and password. Apparently, its in the same database. `git status` indicates that the database has been changed. I am a bit confused here - should I include the database in my `git add` or not? Since I do not know the answer for sure, I am adding it anyway! I am also trying to hold my OCD to `git push` after every commit.    

### Make the app database (model) accessible through the admin access

In `polls/admin.py`, add the code

```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

### Views

In `polls/views.py` add the code (below index which we already added )

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

make changes in `polls/urls.py` so that the pages can be displayed over a browser

```python
rom django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
you can check http://localhost:8000/polls/100/

### I am trying to make the views actually do something instead of returning a predetermined text.

Modify `index` in `polls/views.py` as below, leaving the rest unchanged.

```python
from django.http import HttpResponse
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```

http://localhost:8000/polls/ actually displays the questions. But we need to make it look better

### Introducing templates
inside polls folder, create sub folders `templates/polls` and put an `index.html` file in it
`polls/templates/polls/index.html`

```python
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
