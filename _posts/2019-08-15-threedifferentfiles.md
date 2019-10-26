---
layout: post
title:  "Three different files"
date:   2019-08-15 16:27:19 +0100
categories: free thoughts
---

Do we really need to work on three different files, in order to make a query and show the results in a web page?

The MVC approach changed the way coders made their application. Before MVC taked off it was plenty of spaghetti code everywhere. SQL statements surrounded from JAVA or PHP code all dressed with a generous portion of HTML. It was a mess!

The MVC has given structure to code and helped the devolepers to understand they needed structure and discipline.
The MVC had a very good impact on the developers community.

That sayd I wonder: **"Do we need MVC all the times?"**

The problem I had with frameworks is that they forced me to use MVC. Sometimes it is good to have model view and controller, sometimes it is not.

The main problem I had with MVC is the fact that in order to build a page you need the joined efforts of at least tree files. I say at least because sometimes the view is fractionated in more than one file and sometimes your controller calls more then one model and you find your self having many many files open in your editor and your eyes are jumping from file file trying to understand what is going on in that page that does not respond as it should.

There is so much complexity going on and that complexity is distracting and **distraction lowers your productivity**.

When I develop an application using UD I am not distracted because all the code necessary to define the look and the behaviour of a panel is contained in just one file. A page of code with a piece of business logic defined completely in one script. So **I am able to focus**.

If I am defining a <a href="{{site.baseurl}}/docs/table-page">table</a> I can write the SQL statement at the top of the script, I specify the structure of the table just a litle lower and... that's it.

If I am defining a <a href="{{site.baseurl}}/docs/form">form</a> I start writing the eventual query to pre-fill the fields, I define the structure of the fields and I define the query that updates the database at the bottom.

**I am working on a script at a time.**

That helps me a lot to avoid to loose focus.

I do not want to say that using my approach you can solve any possible problem, I do not want to say that MVC is always a poor approach, I just want to say that sometimes you can use MVC sometimes you can work on a UD json template.

If you are curious about my library you can start from <a href="{{site.baseurl}}/tutorials/crud">the CRUD tutorial</a>.
