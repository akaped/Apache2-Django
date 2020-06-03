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

```bash
sudo apt update && apt upgrade
sudo apt install python3 python3-pip apache2 libapache2-mod-wsgi-py3
```

## 3) Create a virtual environment for python in order to not pollute the system environment.
This is a good practice mostly if your server hosts more than one python applicaiton, my server is dedicated to a one python project only, so I will not do this in production. You can jump this !

```bash
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

```bash
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

Another important line to add is the STATIC_ROOT.
Scroll at the end of the settings.py file and locate STATIC_URL.
STATIC_URL should be equal to static.
Add under it: STATIC_ROOT='/protest/site/public/static'

### 5)  Fire up the Django Web Server, and check if everything is working
```bash
cd /protest/django/protest
python3 manage.py runserver 0.0.0.0:8000
```

You should see some command line output complaining about some migrations, and the webserver running.
If not, you may have introduced some errors along this tutorial, solve this before proceeding.

Open a browser and try to connect to the ip of your webserver and port 8000.
ip_of_server:8000
The Django starting page should be welcoming you!



### 6) Time to switch to Apache!

Type this command to restart the apache2 server we installed before:
`sudo service apache2 restart`
This command is very important and we must fire it everytime we want to apply a new configuration to apache!

Let's open a browser page and point it to:
ip_of_server:80

If Apache2 is running, you should see its default page.

Now we will modify the Apache2 configuration to serve our Django project.
Open 000-default.conf in a text editor:
```bash
nano /etc/apache2/site-available/000-default.conf
```
Delete all its content and replace it with this: (modify the paths to match those of your project)
```apache2
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog /protest/site/logs/error.log
	CustomLog /protest/site/access.log combined

	alias /static /protest/site/public/static
	<Directory /protest/site/public/static>
		Require all granted
	</Directory>

  <Directory /protest/django/protest/protest/>
		<Files wsgi.py>
		Require all granted
		</Files>
  </Directory>

	WSGIDaemonProcess protest python-path=/protest/django/protest python-home=/usr
	WSGIProcessGroup protest
	WSGIScriptAlias / /protest/django/protest/protest/wsgi.py

</VirtualHost>
```

Here the commented version:
```
<VirtualHost * :80>
# In this file we are now defining a new virtualHost that will respond to the requests incoming from port 80 (default http port).

	ServerAdmin webmaster@localhost
  # This server is administred by the user webmaster on localhost.

	DocumentRoot /var/www/html
  # The document root is /var/www/html <-- this is actually not true, but doesn't make any difference.

	ErrorLog /protest/site/logs/error.log
	CustomLog /protest/site/access.log combined
  # We set the error and access logs to be created in the logs folder.

	alias /static /protest/site/public/static
  # this alias tells the server that when a request for the directory static is performed, it should serve the files contained in our static files directory.
	<Directory /protest/site/public/static>
		Require all granted
	</Directory>

  # this is last part regards the configuration of the wsgi script.
  <Directory /protest/django/protest/protest/>
  # set here the directory that contains the wsgi.py script
		<Files wsgi.py>
		Require all granted
		</Files>
  </Directory>

	WSGIDaemonProcess protest python-path=/protest/django/protest python-home=/protest/venv
  # PAY ATTENTION python-home is the main directory where the environment lives!
  # if you created a virtualenvironment the path should point there, otherwise /usr
  WSGIProcessGroup protest
	WSGIScriptAlias / /protest/django/protest/protest/wsgi.py
  # full path to the location of the WSGI script

</VirtualHost>
```

CHECK THE CONFIGURATION:
Apache has a tool to test the server configuration, use it, it will save you from many hours of troubleshooting.
'sudo apachectl configtest'
If it says [OK] you are good to go.

RESTART THE SERVER!
`sudo service apache2 restart`


## 7) TEST it. & Collect Static.

From your browser try again to connect to ip_of_server.
Now you should be able to see again the django welcoming page!.

Last thing:
Django doesn't collect the static files for you!
You need to run:
```bash
python3 manage.py collectstatic
```


That's it!
If you any advice for this tutorial, don't hesitate to contact me!
