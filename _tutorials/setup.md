---
layout: page
title: Setting up a new project
orderfield: 1
---

UD is a PHP library you can use simply importing it in your project using composer.
This is the *composer* file that allows you to take advantage of the library. 

{% highlight json %}
{
    "name": "mycomposername/myprojectname",
    "description": "My description",
    "keywords": ["my keywords"],
    "type": "project",
    "license": "My license",
    "authors": [
        {
            "name": "My name",
            "email": "myemailaddress@example.com"
        }
    ],
    "minimum-stability": "dev",
    "require": {
        "fabiomattei/uglyduckling": "dev-master"
    },
    "autoload": {
        "psr-4": {
            "MyPath\\": "src/"
        }
    }
}
{% endhighlight %}

Once you have done that you need to type:

{% highlight shell %}
composer update
{% endhighlight %}

### Download and startup

A quick way to start is to clone the demo project from <a href="https://github.com/fabiomattei/ud-demo">the github project repository</a> and type:

{% highlight shell %}
composer update
docker-compose build
docker-compose up -d
{% endhighlight %}

You will have a working installation of UD.

### Setting up the database

Once you have done that you can point your browser to http://localhost:8183.

You are going to find an instance of <a href="https://www.phpmyadmin.net/">phpMyAdmin</a> already pointing to your database.

You will be able to import the datamodel from yourlocalfolder/docker/apache/datamodel.sql

### First access to the system

Point your browser to: http://localhost:18080

You will find a log-in page.

On your first install the system defines two users (already part of datamodel.sql):

* **user: _admin_ password: _admin_** this is the user that can create new users and activate or deactivare modules of the system
* **user: _manager_ password: _manager_** this is just a user created in order to have some kind of access to the system

We recommend you to change usernames and password at your first login.

Now you are ready to go.
