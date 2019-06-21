---
layout: post
title:  "The forgotten problem"
date:   2019-06-01 16:27:19 +0100
categories: jekyll update
---

# The forgotten problem

We, as programmers, are funnny creatures.

We like to define our-self as people that loves to solve problems but when we are called to action we prefer to spend our time thinking about technology, frameworks, design patterns, and.... we tend to forget that **we need to solve our clients problems**.

When I receive a document with technical specifications of a system, I start to think about possible implementation solutions, I think a little about the interface, about the necessary libraries and I start to code the business model.

Coding a model is tricky, at the start I am completely focused on the business model and on the reality I am modeling but after a while I start to focus more on the programming language technicalities: hierarchical structure, interfaces, design patterns and I tend to loose the focus I had at the start of the real problem I was going to solve.

Many frameworks promised that they would be helpful speeding up my work and keeping me focused on the business logic but they too often ended up in structures too general that made more complicated to get to a solution.

With time I realized that **to keep my stack of techologies thin was wise**.
I tried to be as close as the nature of the technology I was using I could. I started to be create my prebuilt solutions. I did not want them to be too general, I wanted them general enought in order to simplify my life. When you abstract too much you tend to loose the connection with the reality of things.

After all, when you create a web application you build form that allow the user to POST data and pages to navigate and show that data.

Having this in mind I started from the single page perspective and I wondered what was always the same and what was changing all the time. I started to create models for the most used pages like [tables][docs-datatable], [forms][docs-form] and so on and then I had this idea of putting all things that were generally changing in json files. I built a system that crunching those files was capable of creating a working application and... I ended up building my application framework.

And then I stated to create json definitions for more structures like [information][docs-info] panels and [charts][docs-chartjs].

At the end I started to make containerr in order to group pages elements like [dashboards][docs-dashboard] or [tabbed pages][docs-tabbed-page].

I even tought that was a good idea to define [groups][docs-group] of user using json files.

I  don't know if it was necessary to create one more framework, I even felt guilty for that, but it made my life easier, I am currently using it in order to build more application and it is simplifying my life.

[docs-datatable]:   {{site.baseurl}}/docs/datatable
[docs-form]:        {{site.baseurl}}/docs/form
[docs-info]:        {{site.baseurl}}/docs/info
[docs-chartjs]:     {{site.baseurl}}/docs/chartjs
[docs-dashboard]:   {{site.baseurl}}/docs/dashboard
[docs-tabbed-page]: {{site.baseurl}}/docs/tabbed-page
[docs-group]:       {{site.baseurl}}/docs/group
