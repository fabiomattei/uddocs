---
layout: page
name: Setting the system up
---

# Setting the system up

UD has been designed in order to take advantage of docker so setting it up is relatively easy.

### Download and startup

If you just donwload the master folder from the <a href="https://github.com/fabiomattei/uglyduckling">project repository</a> and you type:

{% highlight shell %}
docker-compose up -d
{% endhighlight %}

You will have a working installation of UD.

### Setting up the database

Once you have done that you can point your browser to http://localhost:9000.

You are going to find an instance of <a href="https://www.phpmyadmin.net/">phpMyAdmin</a> already pointing to your database.

You will be able to import the datamodel from yourlocalfolder/docker/apache/datamodel.sql

### First access to the system

On your first install the systems defines two users:

* admin: this is the user that can create new users and activate or deactivare modules of the system
* manager: this is just a user created in order to have some kind of access to the syste

We recommend to change usernames and password at your first login.


