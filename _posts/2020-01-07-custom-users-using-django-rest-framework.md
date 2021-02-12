---
layout: post
title:  Custom users using Django REST framework
categories: Django
excerpt: he built-in Django User model follows the pattern consisted of username, email and password. In this tutorial you will learn to RESTfully simplify it to just email and password.
---

![](/blog/images/2020-01-07-custom-users-using-django-rest-framework.png)

## Custom users using Django REST framework

I started using Django about a year ago which is way later than it was first released. Back then, in 2005, the predominant user pattern consisted of username, email and password. As web development matured over the years we’ve seen this pattern getting simplified to just email and password. The built-in Django [User model](https://docs.djangoproject.com/en/2.2/ref/contrib/auth/#user-model) follows the old pattern and it can take a few extra steps to change it. It is recommended to roll your custom user model from the start of your project as replacing it in a later phase can be quite a hassle. Don’t just take my word for it, see the [official recommendation](https://docs.djangoproject.com/en/2.2/topics/auth/customizing/#using-a-custom-user-model-when-starting-a-project). **We will go over the steps necessary to get your customized User model set up so that an email address can be used as the primary identifier, RESTfully exposing his endpoints for client apps to use (ReactJS, iOS, Android and other)**.

**Versions used:** Python 3.6.9, Django 3.0, Django REST framework 3.10.3, Django Allauth 0.40.0, Django REST Auth 0.9.5

### Project initialization

It's always a good practice to create isolated Python environments for each project. Here I used [Virtualenv](https://virtualenv.pypa.io/) for that.

Let\`s get things going by quick starting a project named `customuser`:
```bash
mkdir customuser && cd customuser
pip install django
pip install djangorestframework
django-admin startproject customuser .
```

Note that we do not apply any migrations just yet. Now go back to that [official recommendation](https://docs.djangoproject.com/en/2.2/topics/auth/customizing/#using-a-custom-user-model-when-starting-a-project) you didn't bother clicking when reading the intro and find out why.

### Third-party packages

Once the project is created we can begin installing third-party packages that will be working with our custom User model.

```bash
pip install django-allauth
```
[Django Allauth](https://django-allauth.readthedocs.io/en/latest/index.html) handles user registration as well as social authentication. It is also good for email address verification, resetting passwords and much more.

```bash
pip install django-rest-auth
```
[Django REST Auth](https://django-rest-auth.readthedocs.io/en/latest/index.html) conveniently provides API endpoints for user registration, login/logout, password change/reset, social auth, and more.

Make sure to update the `INSTALLED APPS` list in *"settings.py"*:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'rest_auth',
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'rest_auth.registration',
    ...
]
```

### Users app

Now we can create a new app called `users` which will be housing our custom User:

```bash
python manage.py startapp users
```

And add it to the `INSTALLED_APPS` list:
```python
INSTALLED_APPS = [
    ...
    'users',
    ...
]
```

### Custom User manager
The first thing to do is adding a custom [Manager](https://docs.djangoproject.com/en/2.2/topics/db/managers/) which subclasses `BaseUserManager` and instructing it to use an email as the unique identifier instead of a username.

So create a *"managers.py"* file in the *“users”* directory:
```python
from django.contrib.auth.base_user import BaseUserManager
from django.utils.translation import ugettext_lazy as _


class CustomUserManager(BaseUserManager):
    """
    Custom user model manager where email is the unique identifiers
    for authentication instead of usernames.
    """
    def create_user(self, email, password, **extra_fields):
        """
        Create and save a User with the given email and password.
        """
        if not email:
            raise ValueError(_('The Email must be set'))
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save()
        return user

    def create_superuser(self, email, password, **extra_fields):
        """
        Create and save a SuperUser with the given email and password.
        """
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        extra_fields.setdefault('is_active', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError(_('Superuser must have is_staff=True.'))
        if extra_fields.get('is_superuser') is not True:
            raise ValueError(_('Superuser must have is_superuser=True.'))
        return self.create_user(email, password, **extra_fields)

```


### Custom User model

There are two options when creating a custom User model: subclassing `AbstractUser` or subclassing `AbstractBaseUser`

[AbstractUser](https://docs.djangoproject.com/en/2.2/topics/auth/customizing/#django.contrib.auth.models.AbstractUser) is a full User model, complete with fields, like an abstract class so that you can inherit from it and add your own profile fields and methods.

[AbstractBaseUser](https://docs.djangoproject.com/en/2.2/topics/auth/customizing/#django.contrib.auth.models.AbstractBaseUser) only contains the authentication functionality, but no actual fields.

Here we will go with subclassing the `AbstractUser` and making the following changes:
- Remove the `username` field
- Make the `email` field required and unique
- Set the `USERNAME_FIELD` which defines the unique identifier for the `User` model to `email`
- Specify that all objects for the class come from the `CustomUserManager` we created prior
- Add a couple of additional fields for good measure

Here is the code in *"users/models.py"*:
```python
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.utils.translation import ugettext_lazy as _

from .managers import CustomUserManager


class CustomUser(AbstractUser):
    username = None
    email = models.EmailField(_('email address'), unique=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []

    objects = CustomUserManager()

    spouse_name = models.CharField(blank=True, max_length=100)
    date_of_birth = models.DateField(blank=True, null=True)
    

    def __str__(self):
        return self.email
```

### Settings

Adding the following line to the *"settings.py"* file will let Django know to use the new User class:
```python
AUTH_USER_MODEL = 'users.CustomUser'
```

Next, we will add settings to configure `django-allauth`. These are pretty much self-explanatory but you can find a full list with explanations [here](https://django-allauth.readthedocs.io/en/latest/configuration.html).
```python
ACCOUNT_USER_MODEL_USERNAME_FIELD = None
ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_UNIQUE_EMAIL = True
ACCOUNT_USERNAME_REQUIRED = False
ACCOUNT_AUTHENTICATION_METHOD = 'email'
ACCOUNT_EMAIL_VERIFICATION = 'mandatory'
ACCOUNT_CONFIRM_EMAIL_ON_GET = True
ACCOUNT_EMAIL_CONFIRMATION_ANONYMOUS_REDIRECT_URL = '/?verification=1'
ACCOUNT_EMAIL_CONFIRMATION_AUTHENTICATED_REDIRECT_URL = '/?verification=1'

SITE_ID = 1
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

What we have set here:
- usage of email instead of username
- sending the email verification message upon user registration with verification link
- when user clicks on the link his email gets verified and is redirected to `/?verification=1`
- outputting the email messages to console (to send actual emails you must set up Simple Mail Transfer Protocol, simplest way to do this is by using one of the SMTP service providers like [Amazon SES](https://aws.amazon.com/ses/))

After setting the `AUTH_USER_MODEL` we can finally create and apply the migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```

### API

The last thing to do is to include `allauth` and `rest_auth` endpoints for dealing with our custom User in *“urls.py”*:
```python
from allauth.account.views import confirm_email
from django.conf.urls import url
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    url(r'^rest-auth/', include('rest_auth.urls')),
    url(r'^rest-auth/registration/', include('rest_auth.registration.urls')),
    url(r'^account/', include('allauth.urls')),
    url(r'^accounts-rest/registration/account-confirm-email/(?P<key>.+)/$', confirm_email, name='account_confirm_email'),
]
```

This will include:

**User registration**

POST `/rest-auth/registration/`
```json
{
  "email": "test@test.com",
  "password1": "ujmik,ol.",
  "password2": "ujmik,ol."
}
```

This will send out verification email with account confirmation URL looking like this:
`/accounts-rest/registration/account-confirm-email/MQ:1iU6Li:17ruSRLybL38zXvc91no26v2YGw/`

**User login**

Login is denied before user confirms his email.

POST `/rest-auth/login/`
```json
{
  "email": "test@test.com",
  "password": "ujmik,ol."
}
```

**User logout**

POST `/rest-auth/logout/`

**Password change**

User must be logged in to change password (duh…)

POST `/rest-auth/password/change/`
```json
{
  "new_password1": ".lo,kimju",
  "new_password2": ".lo,kimju"
}
```

**Currently logged in user**

GET `/rest-auth/user/`

### Custom User serializer

If you check the response when GETing the currently logged in user you will see something like this:
```json
{
  "pk": 1,
  "username": null,
  "email": "test@test.com",
  "first_name": "",
  "last_name": ""
}
```

As you can see the username field is still there and none of our additional fields are included. This happens because the default User serializer is still being used. A simple solution is to write our own and instruct `rest_auth` to use it.

Create *"serializers.py"* in *"users"* directory:
```python
from rest_framework import serializers

from users.models import CustomUser


class UserSerializer(serializers.ModelSerializer):

    class Meta:
        model = CustomUser
        fields = ['id', 'email', 'first_name', 'last_name', 'spouse_name', 'date_of_birth']
```

And include the following in your *"settings.py"*:
```python
REST_AUTH_SERIALIZERS = {
    'USER_DETAILS_SERIALIZER': 'users.serializers.UserSerializer',
}
```

### Conclusion

These are the basics for RESTfully creating and managing a custom User in Django REST framework. For more customization and options (like social authentication) take a deeper dive into `django-allauth` and `django-rest-auth` packages.
