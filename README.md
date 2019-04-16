# IITG Outlook Oauthv2.0

This Project supports authenticating with Outlook OAuth2 to get a token which is stored in session. 
This token can be used to get user information from outlook like Name, email, Roll Number

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

Create a Virtual Enivronment and install requirements:
```
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Running the demo

A step by step series of examples that tell you how to get a development env running

Migrate to Database & Run the project:
```
./manage.py makemigrations
./manage.py runserver
```

## Add to your project

Copy the authentication package into your own project & add it the installed apps
```
# [your_project]/settings.py

INSTALLED_APPS = [
  ...
    'authentication',
  ...
]
```


Add the authentication urls to your main url.py
```
# [your_project]/urls.py

from django.contrib import admin
from django.conf.urls import include, url
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
    url(r'^authentication/', include('authentication.urls', namespace='authentication')),

]
```

Run your project and go to 127.0.0.1:8000/authentication & test out the authentication


### Authentication views
This will explain the above:
* How to get the Outlook Signin Url? 
* How to redirect to another view after authenticating?
* How to get email, roll number and name from outlook

```
# [your_project]/authentication/views.py

from django.shortcuts import HttpResponse, reverse, redirect

# The Sign In Url to redirect to Outlook Login
from authentication.authhelper import get_signin_url

# Outlook gives us authorization code, we ask for a token from it
from authentication.authhelper import get_token_from_code

# The helper to get data from outlook like roll number, email, name
from authentication.outlookservice import get_me

def home(request):

  # Redirecting to gettoken view after authenticating
  redirect_uri = request.build_absolute_uri(
      reverse('authentication:gettoken'))

  # Building the sig in url
  sign_in_url = get_signin_url(redirect_uri)

  return HttpResponse('<a href="' + sign_in_url + '">Click here to sign in and test outlook OAuth2</a>')


# Get token from code and save it in session

def gettoken(request):

  #################################
  # Set redirect after saving token
  redirect_url = None
  ################################


  # get Token from code
  auth_code = request.GET['code']
  redirect_uri = request.build_absolute_uri(reverse('authentication:gettoken'))
  token = get_token_from_code(auth_code, redirect_uri)
  access_token = token['access_token']

  # Save the token in session
  request.session['access_token'] = access_token

  # redirect_url = request.session.get('redirect_url', None)
  
  if redirect_url is None:

    #####################
    # Get user from token
    user = get_me(access_token)

    return HttpResponse("Token: %s<br>Name: %s<br>Roll Number: %s<br> Mail: %s" % (access_token, user['displayName'], user['surname'], user['mail']))
  else: 
    return redirect(redirect_url)

```

## Built With

* [Virtualenv](http://www.dropwizard.io/1.0.2/docs/) - python virtual environments
* [Django](https://virtualenv.pypa.io/en/latest/) - Web Framework

## Contributing
Create a Pull Request against this repository.

## License

This project is licensed under the MIT License
