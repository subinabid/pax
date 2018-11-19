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

update `polls/views.py`

```python
from django.http import HttpResponse
from django.shortcuts import render
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list,}
    return render(request,'polls/index.html',context)
```

for handling details page, add a `detail.html`
```python
{{ question }}
```

make the changes in `polls/views.html` also
```python
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

It works! I assume `{{question}}` displays only the question text because of
```python
def __str__(self):
      return self.question_text
```
in the model.py

### Removing hard coded URLS in index templates

Replace
```python
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

with

```python
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

`detail` is already defined in `urls.py`

### namespacing

In case there are multiple `detail` URLS across different apps in a project, to make it explicit, include `app_name = 'polls'` in `urls.py` and modify thge template as:

```python
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

### Forms

modify `detail` template as:
```python
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

To handle the submitted data, use `vote` template. Modify `vieww.py` `vote` function":

```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

1. **request.POST** is a dictionary-like object that lets you access submitted data by key name. In this case, request.POST['choice'] returns the ID of the selected choice, as a string. **request.POST** values are always strings.

2. Note that Django also provides request.GET for accessing GET data in the same way – but we’re explicitly using request.POST in our code, to ensure that data is only altered via a POST call.

3. **request.POST['choice']** will raise **KeyError** if choice wasn’t provided in POST data. The above code checks for KeyError and redisplays the question form with an error message if choice isn’t given.

4. After incrementing the choice count, the code returns an **HttpResponseRedirect** rather than a normal HttpResponse. HttpResponseRedirect takes a single argument: the URL to which the user will be redirected (see the following point for how we construct the URL in this case).

5. As the Python comment above points out, you should always return an HttpResponseRedirect after successfully dealing with POST data. This tip isn’t specific to Django; it’s just good Web development practice.

6. We are using the **reverse()** function in the HttpResponseRedirect constructor in this example. This function helps avoid having to hardcode a URL in the view function. It is given the name of the view that we want to pass control to and the variable portion of the URL pattern that points to that view. In this case, using the URLconf we set up in Tutorial 3, this reverse() call will return a string like `'/polls/3/results/'` where the 3 is the value of question.id. This redirected URL will then call the 'results' view to display the final page.

After somebody votes in a question, the `vote()` view redirects to the results page for the question. Let’s write that view:

`polls/views.py`
```python
from django.shortcuts import get_object_or_404, render

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```
`polls/results.html` template
```python
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

PS: Read about `avoiding race condition using F()` later

### Generic Views

Django has built in views to do basic display like showing date from a db. Lets modify `urls.py` and `views.py` to use generic views.

`urls.py`
```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

note:  **<question_id>** is changed to **<pk>**
```

`views.py`
```python
rom django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```

We’re using two generic views here: **ListView** and **DetailView**

PS: read `generic view documentation` later

### Static files

Static files like css and js should be put in `polls/static/polls` folder. All requied files can be added here (eg css, js, images, etc)This is similar to what we do for template files.

For the templates to use these files, add the following to template files

 ```python
 {% load static %}

 <link rel="stylesheet" href="{% static 'polls/style.css' %}">
 ```

`python manage.py runserver` has to be run again to restart the server to account for static files, even if `debug` is on.


### Customizing admin pages

In many cases, we will have to change the way admin page display data.

modify `admin.py` as :

```python
from django.contrib import admin

from .models import Question, Choice

class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']

admin.site.register(Question, QuestionAdmin)
admin.site.register(Choice)
```

A class `QuestionAdmin` is defined ans ia passed as the 2nd arguement to the `register` function, to modify the default display style

When large number of fields are present, the same may be categorized into groups

```python
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]
```

Its better to display the choices along with the questions. Modify as:
``` python
from django.contrib import admin

from .models import Choice, Question


class ChoiceInline(admin.TabularInline):
    model = Choice
    extra = 3


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```

`extra = 3` is for 3 choices

### customizing admin change list

By default, Django displays the **str()** of each object. But sometimes it’d be more helpful if we could display individual fields. To do that, use the list_display admin option, which is a tuple of field names to display, as columns, on the change list page for the object:

```python
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date')
```
