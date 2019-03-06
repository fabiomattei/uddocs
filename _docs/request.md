---
layout: page
name: Request
---

# Request

In a web application it is possible to have two kinds of requests: GET or POST

## GET

"request": {
  "parameters": [
    { "type":"integer", "validation":"required|integer", "name":"id" }
  ]
},

## POST

"request": {
  "postparameters": [
    { "type":"long", "validation":"required|integer", "name":"idform" },
    { "type":"string", "validation":"alphanumerical", "name":"name" }
  ]
},