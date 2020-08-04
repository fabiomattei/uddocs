---
layout: page
title: Setting up a new project
orderfield: 1
---

### Before we start

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

## Starting a demo app

### Download and startup

Before we start you need to be sure you have **composer** and **docker** properly installed on your system. Ulgy Duckling requires **PHP 7.3 or newer** installed on your system but if that is not the case we can take advantage of docker in order to solve the issue.

A quick way to start is to clone the demo project from <a href="https://github.com/fabiomattei/ud-demo">the github project repository</a> 


{% highlight shell %}
git clone https://github.com/fabiomattei/ud-demo
{% endhighlight %}

Now you can type:

{% highlight shell %}
cd ud-demo
composer update
docker-compose build
docker-compose up -d
{% endhighlight %}

You will have a working installation of UD.

If you do not have PHP 7.3 or 7.4 installed on your system you can just *use docker to run composer*. 
Instead of writing *composer update* you are going to type:

{% highlight shell %}
sudo docker run --rm --interactive --tty --volume $PWD:/app composer update
{% endhighlight %}
This command is going to run composer using a docker container.

### Setting up the database

The demo app uses **PHPMyAdmin** to work with the database. 

If you point your browser to **http://localhost:8183**, you are going to find an instance of <a href="https://www.phpmyadmin.net/">phpMyAdmin</a> already pointing to your database.

Now is time di import the data schema located in ud-demo/docker/apache/datamodel.sql

### First access to the system

Point your browser to: http://localhost:18080

You will find a log-in page.

On your first install the system defines two users (already part of datamodel.sql):

* **user: _admin_ password: _admin_** this is the user that can create new users and activate or deactivare modules of the system
* **user: _manager_ password: _manager_** this is just a user created in order to have some kind of access to the system

We recommend you to change usernames and password at your first login.

Now you are ready to go.
