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


## Configure Elastic Beanstalk


1) Install AWS command-line client using `pip install awscli` inside your virtual env
	1) If you encounter problems with installing this, try running `pip install awscli --user` instead
2) Install eb tools using `pip install --upgrade awsebcli`
3) Create your EB environment using `eb init -p python2.7 [PROJECT NAME]`
4) ???


## Django + React

###### Set Up React
1) Move `reactjs.zip` inside your `DJANGO PROJECT FOLDER` so the path looks like `[DJANGO PROJECT FOLDER]/reactjs.zip`
2) Unzip the `reactjs.zip` file and then delete it
3) Move all contents from inside the `reactjs` folder to the outside
	- For example: `reactjs folder `, `fabfile.py`, and the various `webpack` files should all be on the same level with your `DJANGO PROJECT FOLDER`
	- 
###### Configure React

1) In `fabfile.py`, `package.json`, `webpack.base.config.js` and `webpack.prod.config.js`, change the `[YOUR DJANGO APP NAME]` placeholders to your `DJANGO PROJECT NAME`

2) Run `cd [DJANGO PROJECT NAME]` and then run `npm i` to install the node modules





