---
layout: page
name: Group
---

# Structure

{% highlight json %}
{
  "name": "riskmanagergroup",
  "metadata": { "type":"group", "version": "1" },
  "defaultdashboard": "managermaindashboard",
  "home": { "label":"My app", "action":"#" },
  "menu": [
    { "label":"Dashboard", "resource":"managermaindashboard", "icon":"dashboard" },
    {
      "label":"CAR", "icon":"asset",
      "submenu": [
        { "label":"Technical assets", "resource":"UnitList" }
      ]
    },
    {
      "label":"Risk Register", "icon":"asset",
      "submenu": [
        { "label":"Technical assets", "resource":"unitlistdashboard" },
        { "label":"Non-Technical assets", "resource":"NonTechnicalAssetList?level=2" }
      ]
    }
  ]
}
{% endhighlight %}
