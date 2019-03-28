---
layout: page
name: Tabbed page
---

{% highlight json %}
{
  "name": "technicalassetstabbedpage",
  "metadata": { "type":"tabbedpage", "version": "1" },
  "allowedgroups": [ "administrationgroup", "assetspecialistgroup", "centralheadofficegroup", "riskmanagergroup", "operatorgroup" ],
  "title":"Manager dashboard",
  "get": {
    "request": {
      "parameters": []
    }
  },
  "panels":[
    { "title":"Select risk center", "resource":"selectriskcenterform" },
    { "title":"Top 10 risks", "resource":"top10risks" },
    { "title":"Top 10 risks by risk rating", "resource":"top10risksbyriskrating" }
  ]
}
{% endhighlight %}
