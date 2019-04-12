---
layout: page
name: Resource
---

# Resource

Using resources we describe pieces of application to UD.
A resource can be a <a href="{{site.baseurl}}/docs/table-page">table</a> that shows data we get from a database using a query, a resource can be a <a href="{{site.baseurl}}/docs/form">form</a> that is initially filled by data that comes from the database and, after a POST request, saves data back to the database.
A reource can be and <a href="{{site.baseurl}}/docs/info">info</a> panel that load data from the database and shows that data to the user, or can be a <a href="{{site.baseurl}}/docs/chartjs">chart</a> that shows visually data calculated by formulas applied to the database. 

Even a set of queries we apply to the database using a <a href="{{site.baseurl}}/docs/logics">logic</a> is a resource.

Using resources we describe the system piece by piece to UD.

Designing a system, in UD terms means to create a set of json file or resources we provide to the system in order to describe the interface and the logics.

Each reource is a json file with a unique name, a unique path and a specific type. It is listed in a <a href="{{site.baseurl}}/docs/jsonindex">Index</a> and contains all related information to describe part of the software system.
