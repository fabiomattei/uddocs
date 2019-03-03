---
layout: page
name: Transaction
---

```
#!json

{ 
  "name": "deletereportv1",
  "metadata": { "type":"logic", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"integer", "validation":"required|integer", "name":"id" }
      ]
    },
    "logics": [
      {
        "sql":"DELETE from requestv1 WHERE id=:id;",
        "parameters":[
          { "placeholder": ":id", "getparameter": "id" }
        ]
      }
    ]
  }
}
```