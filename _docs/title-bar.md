---
layout: page
name: Title bar
---

```
#!json
{
  "name": "unitlisttitlebar",
  "metadata": { "type":"titlebar", "version": "1" },
  "allowedgroups": [ "centralheadofficegroup" ],
  "get": {
    "request": {
      "parameters": [
      ]
    },
    "titlebar": {
      "title": "",
      "titleresources": [
        { "resource":"unitbreadcrumbs" }
      ],
      "actions": [
        { "label": "New Risk Center Unit", "resource": "newriskcenterunit", "buttoncolor":"btn-success", "outline": false, "parameters":[] }
      ]
    }
  }
}
```