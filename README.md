ngo React AWS(Elastic Beanstalk) Seed Project

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

## Configure Elastic Beanstalk


1) Install AWS command-line client using `pip install awscli` inside your virtual env
    1) If you encounter problems with installing this, try running `pip install awscli --user` instead
2) Install eb tools using `pip install --upgrade awsebcli`
3) Create your EB environment using `eb init -p python2.7 [PROJECT NAME]`
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




