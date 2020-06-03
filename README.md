# Apache2-Django-Tutorial

This is a tutorial for setting up an Apache2 server that is able to serve a Django project for production purposes.
If you are reading this is probably because you are finally ready to take your Django project to the next level!
Despite the fact that the website of Django offers some guidance in this [process](https://docs.djangoproject.com/en/3.0/howto/deployment/wsgi/modwsgi/) I found it poor in explaining how to do this step by step. In order to do set it up correctly I had to read many online tutorials and the apache2 documentation.
The tutorial will start a django project, install apache2 and its configuration to serve the django website and its static files.

## Advices.
This tutorial is based on the following software packages/OS.
Different software versions may differ.

OS: Debian GNU/Linux 10 (buster)
Apache2
Python3 3.7.3
Django 3.0.7

I'm calling this project: **protest** (as production-test).


## 1) Set-Up your folders.

The first thing to get in place is where your Django project will live.
We need a folder that will be accessible to any user, Apache2 works with its own generated user.
A good solution is to deploy your project in the root directory of the system.

The structure will be the following:
```
/protest/
|_ site/
|  |_ logs/ #all apache logs
|  |_ public/
|    |_ static/ #all static files
|_ django/
|    |_ ... #all django files
|_ auth
```

```bash
cd /
mkdir protest
cd protest
mkdir logs public django auth
cd public
mkdir static
cd /protest
```

## 2) Install required software packages & update your server

This command is easy to understand on its own, we are going to use apt packet manager to install apache2 and python3 as well as some other needed things.

```
sudo apt update && apt upgrade
sudo apt install python3 python3-pip apache2 libapache2-mod-wsgi-py3
```

## 3) Create a virtual environment for python in order to not pollute the system environment.
This is a good practice mostly if your server hosts more than one python applicaiton, my server is dedicated to a one python project only, so I will not do this in production. You can jump this !

```
cd /protest
sudo pip3 install virtualenv
virtualenv venv -p python3
```

Once the virtualenvironment is deployed let's activate it:
`./venv/bin/activate`

## 4) Install Django & setup the config.py
You are now inside the virtual environment you created, now you need to add to this environment all the python packages you need:
Along with django, I usually install also [django-extensions](https://django-extensions.readthedocs.io/en/latest/).
Django-extensions are a collection of useful extensions for Django that include management/administrative and much more.

```
pip3 install django django-extensions
```

Now move to the directory of your project where the settings lives:
in my case: /protest/django/protest/protest/settings.py

Open the file with a text editor 'nano settings.py'
And add your server ip to ALLOWED_HOSTS=[]
Example: ALLOWED_HOSTS=['192.168.1.33']
A little trick i use during development when I need to test the project on different machines is to allow all hosts!
This shouldn't be done in production!
ALLOWED_HOSTS=[' * ']

### 5)  Fire up the Django Web Server, and check if everything is working
```
cd /protest/django/protest
python3 manage.py runserver 0.0.0.0:8000
```

You should see some command line output complaining about some migrations, and the webserver running.
If now, you may have introduced some errors along this tutorial, solve this before proceeding.


### 6)
