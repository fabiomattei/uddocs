---
layout: page
name: Dashboard
---

# Structure


```
#!json
{
  "name": "centralheadofficedashboard",
  "metadata": { "type":"dashboard", "version": "1" },
  "allowedgroups": [ "centralheadofficegroup" ],
  "title":"Manager dashboard",
  "get": {
    "request": {
      "parameters": []
    }
  },
  "panels":[
    { "title":"My chart", "resource":"selectriskcenterform", "row":"1", "width":"12" },
    { "title":"My chart", "resource":"top10risks", "row":"2", "width":"6" },
    { "title":"My table", "resource":"top10risksbyriskrating", "row":"2", "width":"6" },
    { "title":"My table", "resource":"top10rootcauses", "row":"3", "width":"6" },
    { "title":"My table", "resource":"top10consequences", "row":"3", "width":"6" },
    { "title":"My table", "resource":"unitheatmap", "row":"4", "width":"6" }
  ]
}
```