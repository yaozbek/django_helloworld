###  Initial Set Up
To start navigate to a new directory on your computer. For example, *django_helloworld* folder can create on the Desktop with the following commands.
```
$ cd ~/Desktop
$ mkdir django_helloworld
$ cd django_helloworld
```
Make sure you’re not already in an existing virtual environment at this point. If you see text in parentheses () before the dollar sign $ then you are. To exit it, type *exit* and hit *Return* . The parentheses should disappear which means that virtual environment is no longer active.

*pipenv* is used to create a new virtual environment, install Django and then activate it.

```
$ pipenv install django==2.1
$ pipenv shell
```

Create a new Django project called *helloworld_project* making sure to include the period . at the end of the command so that it is installed in current directory.

```
(django_helloworld) $ django-admin startproject helloworld_project .
```
```
(django_helloworld) $ tree
.
├── helloworld_project
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
├── Pipfile
└── Pipfile.lock
```

The *settings.py* file controls your project’s settings, *urls.py* tells Django which pages to build in response to a browser or URL request, and *wsgi.py* , which stands for [web server gateway interface](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface), helps Django serve your eventual web pages. The last file *manage.py* is used to execute various Django commands such as running the local web server or creating a new app. Django comes with a built-in web server for local development purposes. *manage.py* can start with the *runserver* command.
```
(django_helloworld) $ python manage.py runserver
```
If you visit http://127.0.0.1:8000/ you should see the following image:

###  Create an app

Django uses the concept of projects and apps to keep code clean and readable. A single Django project contains one or more apps within it that all work together to power a web application. This is why the command for a new Django project is *startproject*! For example, a real-world Django 
e-commerce site might have one app for user authentication, another app for payments, and a third app to power item listing details. Each focuses on an isolated piece of functionality.

You need to create your first app which you’ll call *pages*.
```
(django_helloworld) $ python manage.py startapp pages
```
If you look again inside the directory with the *tree* command you’ll see Django has created a *pages* directory with the following files:
```
(django_helloworld) $ tree
.
├── helloworld_project
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
├── pages
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── Pipfile
└── Pipfile.lock
```
Let’s review what each new *pages* app file does:
- *admin.py* is a configuration file for the built-in Django Admin app
- *apps.py* is a configuration file for the app itself
- *migrations/* keeps track of any changes to your *models.py* file so your
database and *models.py* stay in sync
- *models.py* is where you define your database models, which Django automatically translates into database tables
- *tests.py* is for your app-specific tests
- *views.py* is where you handle the request/response logic for your web app

Even though your new app exists within the Django project, Django doesn’t “know” about it until you explicitly add it. In your text editor open the *settings.py* file and scroll down to *INSTALLED_APPS* where you’ll see six built-in Django apps already there. Add your new *pages* app at the bottom.
```
# helloworld_project/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
You should always add your own local apps at the bottom because Django will read your *INSTALLED_APPS* in top down order. Therefore the internal *admin* app is loaded first, then *auth* , and so on. You want the core Django apps to be available since it’s quite likely your own apps will rely on their functionality.

Another thing to note is you might be wondering why you can’t just list the app name *pages* here instead of the much longer *pages.apps.PagesConfig*? The answer is that Django creates an apps.py file with each new application and it’s possible to add additional information there, especially with the [Signals framework](https://docs.djangoproject.com/en/2.1/topics/signals/) which is an advanced technique. For your relatively basic
application using pages instead would still work, but you’d miss out on additional options and so as a best practice, always use the full app config name like *pages.apps.PagesConfig*.

### Views and URLConfs

In Django, *Views* determine **what** content is displayed on a given page while *URLConfs* determine **where** that content is going. 

When a user requests a specific page, like the homepage, the URLConf uses a [regular expression](https://en.wikipedia.org/wiki/Regular_expression) to map that request to the appropriate view function which then returns the correct data. 

In other words, your *view* will output the text “Hello, World” while your *url* will ensure that when the user visits the homepage they are redirected to the correct view. 

This interaction is frequently **very confusing** so let’s map out the order of a given HTTP request/response cycle. When you type in a URL, such as "https://google.com" the first thing that happens within your Django project is a URLpattern is found that matches the homepage. The URLpattern specifies a view which then determines the content for the page (usually from the database) and a template for styling. The end result is sent back to the user as an HTTP response.


Django request/response cycle
```
URL -> View -> Model (typically) -> Template
```
Let’s start by updating the *views.py* file in your *pages* app to look as follows:
```
# pages/views.py
from django.http import HttpResponse

def homePageView(request):
    return HttpResponse('Hello, World!')
```
Basically you’re saying whenever the view function *homePageView* is called, return the text “Hello, World!” More specifically, you’ve imported the built-in [HttpResponse](https://docs.djangoproject.com/en/2.1/ref/request-response/#django.http.HttpResponse) method so you can return a response object to the user. You’ve created a function called *homePageView* that accepts the *request* object and returns a response with the string *Hello, World!*. Now you need to configure your urls. Within the *pages* app, create a new *urls.py* file.

```
(django_helloworld) $ touch pages/urls.py
```
Then update it with the following code:
```
# pages/urls.py
from django.urls import path

from .views import homePageView

urlpatterns = [
    path('', homePageView, name='home')
]
```
On the top line you import *path* from Django to power your *URLpattern* and on the next line you import your views. By using the period *.views* you reference the current directory, which is your pages app containing both *views.py* and *urls.py* . Your URLpattern has three parts:
- a Python regular expression for the empty string ''
- specify the view which is called *homePageView*
- add an optional URL name of '*home*'

In other words, if the user requests the homepage, represented by the empty string '' then use the view called *homePageView*.

The last step is to configure your project-level *urls.py* file since it’s common to have multiple apps within a single Django project, so they each need their own route.

> “Project-level” means the topmost, parent directory of an application. In this case where both the *helloworld_project* and *pages* app folders exist. Once you are *inside* a specific app you are “app-level.”

```
# helloworld_project/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('pages.urls')),
]
```
You’ve imported *include* on the second line next to *path* and then created a new URLpattern for your *pages* app. Now whenever a user visits the homepage at / they will first be routed to the *pages* app and then to the *homePageView* view.

It’s often confusing that you don’t need to import the *pages* app here, yet you refer to it in your URLpattern as *pages.urls*. The reason you do it this way is that the method *django.urls.include()* expects us to pass in a module, or app, as the first argument. So without using *include* you would need to import your *pages* app, but since you do use *include* you don’t have to at the project level!

**Hello, world!**
To confirm everything works as expected, restart your Django server:
```
(django_helloworld) $ python manage.py runserver
```
If you refresh the browser for http://127.0.0.1:8000/ it now displays the text “Hello, world!”
