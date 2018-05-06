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
