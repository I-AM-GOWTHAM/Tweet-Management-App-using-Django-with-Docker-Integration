# project folder creation
```
mkdir Django-full-stack-project
cd Django-full-stack-project
```
creating virtual env as .venv
```
python3 -m venv .venv  
cd .venv/scripts
activate
cd ../..
```
# Install Django
```
pip install django  #intall django
pip install --upgrade pip  #upgrade pip optional
```
//creating requirements.txt for packages versions
```pip freeze > requirements.txt```
//creating django project with name tweet_project then run it 
```
django-admin startproject tweet_project
cd tweet_project
```
//run the project
python manage.py runserver 
//make migrations
```
python manage.py makemigrations  
python manage.py migrate
```
//creating super user and create credentials username password
```python manage.py createsuperuser```  
//MODIFY SETTING.PY
``` 
import os 
#ADDED CODE FOR WHERE THE MEDIA FILE WILL BE STORED
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
#ADDED CODE FOR WHERE THE STATIC FILES WILL BE STORED
STATIC_URL = 'static/'
STATICFILES_DIRS = [os.path.join(BASE_DIR,'static')]
```
//MODIFY url.py and add commands like below
```
from django.conf import settings
from django.conf.urls.static import static
urlpatterns = [
    path('admin/', admin.site.urls),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
//creating django app with name tweet_app
```python manage.py startapp tweet_app ```

//goto that app and create urls.py file and add basic code
//now create a view in views.py in app
//app/views.py
```
def index(request):
    return render(request,'index.html')
```
create path to that index view as below
//app/urls.py
```
from django.urls import path 
from . import views
urlpatterns = [
    path('', views.index, name='index'),
]
//add 'tweet' app name in installed app in settins.py
INSTALLED_APPS = [
    ....,
    'tweet_app',
]
```
//create templates folder app/templates/index.html/basic code
```
TEMPLATES = [
    {
        ....,
        'DIRS': [os.path.join(BASE_DIR,'templates')],   //adding templates folder for templates for each app
        ....,
    },
]
```

//in project urls.py add app.views.urls
```
from django.urls import path,include //adding include
urlpatterns = [
    ....,
    path('tweet/', include('tweet_app.urls')),  //connecting app.urls to the project urls
]
```
//run the manage.py you can see index page text as result
//UPTO NOW you created a project and an app you connected app to project in urls.py and settings.py 
//CREATED A INDEX VIEW IN APP AND CONNECT WITH URL CREATED TEMPLATES FOLDER written code for index.html so upto now i hope its clear 
//TO PROVIDE COMMON TEMPLATES CREATE FOLDER TEMPLATES IN mainfolder(basefolder)/templates also
//mainfolder(basefolder)/templates/layout.html
```
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" 
    rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    <title>
        {% block title%}layout
        {% endblock %}
    </title>
</head>
<body>
    <div class="container">
        {% block content %}
        {% endblock %}
    </div>
</body>
</html>
```

//DELETE THE HTMLCODE IN index.html 
```
{% extends "layout.html" %}
{% block title%}layout page{% endblock %}
{% block content%}
<h1 class="text-center mt-4">django project INDEX PAGE CONTENT</h1>
{% endblock %}
```
//install pillow for photos upload tweet purpose
```
python -m pip install pillow
```
and then update the pillow version to the requirements.txt by the command ```pip freeze>requirements.txt```
make sure you are at the mainfolder(basefolder)/ when you running the freeze command beacuse requirements.txt is there only

//NOW CREATE MODELS IN app/models.py
```
from django.db import models
from django.contrib.auth.models import User
# Create your models here.
class tweet(models.Model):
    user = models.ForeignKey(User,on_delete=models.CASCADE)   
    tweet = models.TextField(max_length=240)
    photo = models.ImageField(upload_to='photos/',blank=True,null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f'{self.user.username} - {self.text[:10]}'   //this is to diplay username - subject at admin portal
```

//migrate the tweet model by makemigrations
```
python manage.py makemigrations
python manage.py migrate  -> done
```
//whenever you created model you need to register in app/admin.py
```
from django.contrib import admin
from . models import Tweet
# Register your models here.
admin.site.register(Tweet)
```

//CREATIN FORMS.PY 
create app/forms.py
```
from django import forms
from .models import Tweet

class TweetForm(forms.ModelForm):
    class Meta:
        model = Tweet
        fields = ['text','photo']

```

//now create views to render the tweetlist and forms in html
app/views.py
```
from .models import Tweet
from .forms import TweetForm
def tweet_list(request):
    tweets = Tweet.objects.all().order_by('-created_at')
    return render(request,'tweet_list.html',{'tweets':tweets})
```
//now create views for create_tweet,tweet_delete,tweet_edit
```
from django.shortcuts import render,redirect
from .models import Tweet
from .forms import TweetForm
from django.shortcuts import get_list_or_404
# Create your views here.

def index(request):
    return render(request,'index.html')

def tweet_list(request):
    tweets = Tweet.objects.all().order_by('-created_at')
    return render(request,'tweet_list.html',{'tweets':tweets})


def tweet_create(request):
    if request.mehtod == 'POST':
        form = TweetForm(request.POST, request.FILES)
        if(form.is_valid()):
            tweet = form.save(commit=False)
            tweet.user = request.user
            tweet.save()
            return redirect('tweet_list.html')
    else:
        form = TweetForm()
    return render(request,'tweet_form.html',{'form':form})

def tweet_edit(request,tweet_id):
    tweet = get_list_or_404(Tweet,pk = tweet_id)
    if request.method == 'POST':
        form = TweetForm(request.POST, request.FILES,instance=tweet)
        if(form.is_valid()):
            tweet = form.save(commit=False)
            tweet.user = request.user
            tweet.save()
            return redirect('tweet_list.html')
    else:
        form = TweetForm(instance=tweet)
    return render(request,'tweet_form.html',{'form':form})


def tweet_delete(request,tweet_id):
    tweet = get_list_or_404(Tweet,pk = tweet_id, user= request.user)
    if request.method == 'POST':
        tweet.delete()
        return redirect('tweet_list.html')
    return render(request,'tweet_confirm_delete.html',{'tweet':tweet})

```


//now templates tweet_list.html,tweet_form.html,tweet_confirm_delete.html
/app/templates/tweet_list.html
```
{% extends "layout.html" %}
{% block title%}layout page{% endblock %}

{% block content%}
<h1 class="text-center mt-4">django project TWEET LIST PAGE CONTENT</h1>

<a href="{% url 'tweet_create' %}" class="btn btn-primary mb-4">Create a tweet</a>
<div class="container row gap-3">
    {% for tweet in tweets  %}
    <div class="card" style="width: 18rem;">
        <img src="{{tweet.photo.url}}" class="card-img-top" alt="...">
        <div class="card-body">
          <h5 class="card-title">{{tweet.user.username}}</h5>
          <p class="card-text">{{tweet.text}}</p>
          <a href="{% url 'tweet_edit' tweet.id %}" class="btn btn-primary">Edit</a>
          <a href="{% url 'tweet_delete' tweet.id %}" class="btn btn-danger">Delete</a>
        </div>
      </div>
    {% endfor %}
</div>

{% endblock %}
```
app/templates/tweet_form.html
```
{% extends "layout.html" %}
{% block title%}layout page{% endblock %}

{% block content%}
<h1 class="text-center mt-4">django project TWEET FORM PAGE CONTENT</h1>
<h2>{% if form.instance.pk %}
    Edit Tweet 
    {% else %} 
    Create Tweet
    {% endif %}
    <form method="POST" enctype="multipart/form-data" class="form">
        {% csrf_token%}
        {{form.as_p}}
        <button class="btn btn-warning" type="submit">Submit</button>
        <a href="{% url 'tweet_list' %}">Back to tweet list</a>
    </form>
</h2>
{% endblock %}
```
app/tweet_confirm_delete.html
```
{% extends "layout.html" %}
{% block title%}layout page{% endblock %}

{% block content%}
<h1 class="text-center mt-4">Are you sure that you want to delete tweet?</h1>
<form method="POST">
    {% csrf_token %}
    <button class="btn btn-danger">Delete</button>
    <a class="btn btn-success" href="{% url 'tweet_list' %}">Cancel</a>
</form>
{% endblock %}
```
//NOW ADD USER REGISTRATION,LOGIN,LOGOUT FEATURE
in app/forms.py
```
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
class UserRegistrationForm(UserCreationForm):
    email = forms.EmailField()
    class Meta:
        model = User
        fields = ('username','email','password1','password2')
```
in app/views.py
```
from django.contrib.auth.decorators import login_required
@login_required  #put this on top of function in which there is login requires FEATURE

@login_required
def tweet_create(request):

@login_required
def tweet_edit(request,tweet_id):

@login_required
def tweet_delete(request,tweet_id):
```
//after that write views for registerations
```
from .forms import TweetForm,UserRegistrationForm
from django.shortcuts import get_object_or_404
from django.contrib.auth import login
def register(request):
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)
        if form.is_valid():
            user = form.save(commit=False)
            user.set_password(form.cleaned_data['password1'])
            user.save()
            login(request,user)
            return redirect('tweet_list')
    else:
        form = UserRegistrationForm()
    return render(request,'registration/register.html',{'form':form})
```
//now create registration folder in mainfolder/templates
//create logged_out.html,login.html,register.html
//mainfolder/templates/registeration/register.html
```
{% extends "layout.html" %}
{% block title%}layout page{% endblock %}
{% block content%}
<h1 class="text-center mt-4">django project TWEET FORM PAGE CONTENT</h1>
<h2>Register Form</h2>
<form method="POST" class="form">
    {% csrf_token%}
    {{form.as_p}}
    <button class="btn btn-primary">Register</button>
</form>
{% endblock %}
```

//mainfolder/templates/registeration/logout.html
```
{% extends "layout.html" %}
{% block title%}layout page{% endblock %}
{% block content%}
<h2>Login Form</h2>
<form method="POST" class="form">
    {% csrf_token%}
    {{form.as_p}}
    <button class="btn btn-warning">Login</button>
    <p>Dont have an account? <a href="{% url 'register' %}">Register here</a></p>
</form>
{% endblock %}
```
//mainfolder/templates/registeration/logout.html
```
{% extends "layout.html" %}
{% block title%}layout page{% endblock %}
{% block content%}
<h2>Logged Out</h2>
<p>You have been logged out <a href="{% url 'login' %}">Login here</a></p>
{% endblock %}

now add path for register in project/urls.py
from django.urls import path,include
from django.conf import settings
from django.conf.urls.static import static
from django.contrib.auth.urls import views as auth_views  //added line
urlpatterns = [
    path('admin/', admin.site.urls),
    path('tweet/', include('tweet_app.urls')),
    path('accounts/', include('django.contrib.auth.urls')),   //added line
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
//in settings.py add url for login,login_redirect,logout_redirect
```
LOGIN_URL = '/accounts/login'
LOGIN_REDIRECT_URL = '/tweet/'
LOGOUT_REDIRECT_URL = '/tweet/'
```
//modify some code in mainfolder/templates/layout.html the final code for layout.html below
```
{% load static %}
<!DOCTYPE html>
<html lang="en"  data-bs-theme="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" 
    rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    <title>
        {% block title%}layout
        {% endblock %}
    </title>
    <style>
        
    </style>
</head>
<body>

    <nav class="navbar navbar-expand-lg bg-body-tertiary">
        <div class="container-fluid">
          <a class="navbar-brand" href="#">TweetBar</a>
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
              <li class="nav-item">
                <a class="nav-link active" aria-current="page" href="#">Home</a>
              </li>
              <li class="nav-item">
                <a class="nav-link" href="#">Link</a>
              </li>
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                  Dropdown
                </a>
                <ul class="dropdown-menu">
                  <li><a class="dropdown-item" href="#">Action</a></li>
                  <li><a class="dropdown-item" href="#">Another action</a></li>
                  <li><hr class="dropdown-divider"></li>
                  <li><a class="dropdown-item" href="#">Something else here</a></li>
                </ul>
              </li>
              <li class="nav-item">
                <a class="nav-link disabled" aria-disabled="true">Disabled</a>
              </li>
            </ul>
            <form class="d-flex" role="search">
              <input class="form-control me-2" type="search" placeholder="Search" aria-label="Search">
              <button class="btn btn-outline-success" type="submit">Search</button>
            </form>
            <a href="{% url 'tweet_list' %}" class="btn btn-primary mx-2">Tweet home</a>
            {% if user.is_authenticated %}
            <form method='POST' action="{% url 'logout' %}">
              {% csrf_token %}
              <button class="btn-btn-danger" type="submit">Logout</button>
            </form>
            {%else%}
            <a href="{% url 'register' %}" class="btn btn-primary mx-2">Register</a>
            <a href="{% url 'login' %}" class="btn btn-success mx-2">Login</a>
            {% endif %}
          </div>
        </div>
      </nav>

    <div class="container">
        {% block content %}
        {% endblock %}
    </div>
</body>
</html>
```


RUN THE CODE ...
SUCCESSFULLY DEVELOPED A REAL TIME TWEET APPLICATION USING Django

