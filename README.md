# Django React AWS(Elastic Beanstalk) Seed Project

This guide will help you setup a Django app to use React.js via templates and deploy to AWS using their Elastic Beanstalk service.

## Set Up Django Project


- Follow prerequisites from http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-django.html

###### Create Python Virtual Environment for Django

1) `virtualenv eb-virt`
2) `source eb-virt/bin/activate`

###### Install Requirements

1) `pip install -r requirements.txt`

###### Create Django Project

1) `django-admin startproject [PROJECT NAME]`
2) `python [PROJECT NAME]/manage.py migrate`

###### Create Django App

1) Run `django-admin.py startapp [APP NAME]` inside your `DJANGO PROJECT FOLDER`
2) Add app to `settings.py` in the `INSTALLED_APPS` section as `[DJANGO PROJECT NAME].[APP NAME],`

###### Configure Django Project
1) Create `static` and `templates` folder inside the `DJANGO PROJECT FOLDER` (same level as `DJANGO APP`)

2) Change the `DIRS` property of the `TEMPLATES` variable to be 

```
'DIRS': [os.path.join(BASE_DIR, '[DJANGO PROJECT NAME]/templates')],
```

3) Add this underneath the `TEMPLATES` variable: 

```
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, '[DJANGO PROJECT NAME]/static'),
]
```

4) Add this on top of the `STATIC_URL`    variable: 
```
STATIC_ROOT = os.path.join(BASE_DIR, "..", "www", "static")
```

###### Configure Other Settings

Paste the code below in your `settings.py` file to use AWS S3 to store static files (such as images)

```
AWS_STORAGE_BUCKET_NAME = '[YOUR S3 BUCKET NAME]'
MEDIA_URL = 'http://%s.s3.amazonaws.com/uploads/' % AWS_STORAGE_BUCKET_NAME
DEFAULT_FILE_STORAGE = "storages.backends.s3boto.S3BotoStorage"
```

Change `ALLOWED_HOSTS` to
```
ALLOWED_HOSTS = ['localhost', '.elasticbeanstalk.com']  
```

## Configure Elastic Beanstalk


1) Install AWS command-line client using `pip install awscli` inside your virtual env
    1) If you encounter problems with installing this, try running `pip install awscli --user` instead
2) Install eb tools using `pip install --upgrade awsebcli`
3) Make the following changes to your `settings.py` file:
- Replace the `DATABASES` variable with:

```
if 'RDS_DB_NAME' in os.environ:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': os.environ['RDS_DB_NAME'],
            'USER': os.environ['RDS_USERNAME'],
            'PASSWORD': os.environ['RDS_PASSWORD'],
            'HOST': os.environ['RDS_HOSTNAME'],
            'PORT': os.environ['RDS_PORT'],
        }
    }
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': '[LOCAL DATABASE NAME]',
            'USER': '[LOCAL DATABASE USER]',
            'PASSWORD': '',
            'HOST': 'localhost',
            'PORT': '5432',
        }
    }
```

This above code checks the environment variables to see if it is `local` or `production`. If it is not the `local` environment then it will use the AWS RDS variables to connect to the `production` database instead of the `local` database.

NOTE: You'll have to download Postgres and create the local database yourself via commandline

- Copy this code below the code from the step above

```
# AWS EB Settings
AWS_QUERYSTRING_AUTH = False
AWS_ACCESS_KEY_ID = '[YOUR AWS ACCESS KEY ID]'
AWS_SECRET_ACCESS_KEY = '[YOUR AWS SECRET ACCESS KEY]'
```

3) Initialize your EB environment using `eb init -p python2.7 [PROJECT NAME]`

4) Create your EB environment using `eb create eb-virt` (this will launch and create your elastic beanstalk environment)

	- If you get an error like `Error: pg_config executable not found` in your `eb logs` then: 


4) `STILL IN PROGRESS`


## Django + React

###### Set Up React
1) Move `reactjs.zip` inside your `DJANGO PROJECT FOLDER` so the path looks like `[DJANGO PROJECT FOLDER]/reactjs.zip`
2) Unzip the `reactjs.zip` file and then delete it
3) Move all contents from inside the `reactjs` folder to the outside
    - For example: `reactjs` folder, `fabfile.py`, and the various `webpack` files should all be on the same level with your `DJANGO PROJECT FOLDER`
4) In the same level as the `reactjs` folder, create a file called `.babelrc` whose contents are:

```
{
  "presets": ["es2015", "react", "stage-0"],
  "plugins": [
    ["transform-decorators-legacy"],
  ]
}
```

###### Configure React

1) In `fabfile.py`, `package.json`, `webpack.base.config.js` and `webpack.prod.config.js`, change the `[YOUR DJANGO APP NAME]` placeholders to your `DJANGO PROJECT NAME`

    - NOTE: change the output path in `webpack config` files to be `./[DJANGO PROJECT NAME]/static/bundles/prod/` instead of it containing any `DJANGO PROJECT` specific folder names
2) Run `cd [DJANGO PROJECT NAME]` and then run `npm i` to install the node modules


###### Configure Django for React

Add this to your `settings.py`
```
WEBPACK_LOADER = {
    'DEFAULT': {
        'BUNDLE_DIR_NAME': 'bundles/local/',  # end with slash
        'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats-local.json'),
    }
}
```

Add this to `INSTALLED_APPS`: `'webpack_loader',`


###### Using React with Django Templates

To use React inside Django templates you have to do something like this (example):

```
{% load render_bundle from webpack_loader %}

<div id="App"></div>

{% render_bundle 'vendors' %}
{% render_bundle 'App' %}
```

The parts of the example above that will be different for you will be: 
`<div id="App"></div>` and `{% render_bundle 'App' %}`. 

If you check your `webpack.base.config.js` file, you will see that `module.exports` has an object called `entry`. This object specifies what React component(s)/container(s) will have bundles created for them. 

These React component(s)/container(s) are the top-most and almost all other React components/containers will be their children. You can have multiple different entry points thus, separating your React code into multiple different bundles that you are free to use in whichever Django template you like.

Since we have an `App.jsx` file in our `reactjs` folder, we have created an entry point called `App`. We use `{% render_bundle 'App' %}` to use the bundle specifically generated for `App`.


## Clean Up

1) Replace placeholder with your `DJANGO PROJECT NAME` in `.gitignore` to ignore the `node_modules` folder


## Questions
Please email neelk07@gmail.com


## Helpful Links

- http://www.trysudo.com/deploying-django-app-on-aws-using-elastic-beanstalk/
- http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-django.html

## Random Errors

```
sudo yum install libtiff-devel libjpeg-devel libzip-devel freetype-devel \
    lcms2-devel libwebp-devel tcl-devel tk-devel
```

## Deploying to Heroku 

1) Move `requirements.txt` inside the outer `DJANGO PROJECT FOLDER` (so it is on the same level as the `manage.py`

2) Create `Procfile` with the following contents:
```
web: gunicorn {{ project_name }}.wsgi
```
3) Initialize `.git` on the same level as `manage.py`

4) Create local database by logging into Postgres and running `CREATE DATABASE [NAME];`

5) Followed https://github.com/SamSamskies/django-webpack-heroku-example

6) Add Postgres add-on for Django by running 
	```
	heroku addons:create heroku-postgresql:hobby-dev
	```
7) Install the necessary `python` libraries:
	```
    pip install dj-database-url
    pip freeze > requirements.txt
	```
7) Change your `settings.py` file:
	```
    import dj_database_url
    DATABASES['default'] =  dj_database_url.config()
    ```
	- https://devcenter.heroku.com/articles/heroku-postgresql#connecting-in-python 
