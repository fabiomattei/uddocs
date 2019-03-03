---
layout: page
name: Chartjs
---

# Structure
```
#!json
{
  "name": "top10consequences",
  "metadata": { "type":"chartjs", "version": "1" },
  "allowedgroups": [ "administrationgroup", "assetspecialistgroup", "centralheadofficegroup", "riskmanagergroup", "operatorgroup" ],
  "get": {
    "request": {
      "parameters": []
    },
    "query": {
      "sql": "SELECT TRUNCATE(SUM((MS.mv_value + MT.mv_value + MF.mv_value + MR.mv_value + ME.mv_value) * ML.ml_likelihood),2) AS monetizedrisk, RC.c_name, r_consequencesid   FROM risk AS R   JOIN monetizedvalues AS MS ON MS.mv_id = R.r_safety   JOIN monetizedvalues AS MT ON MT.mv_id = R.r_technical   JOIN monetizedvalues AS MF ON MF.mv_id = R.r_finantial   JOIN monetizedvalues AS MR ON MR.mv_id = R.r_reputation   JOIN monetizedvalues AS ME ON ME.mv_id = R.r_environmental   JOIN monetizedlikelihood AS ML ON ML.ml_id = R.r_likelihood JOIN consequences AS RC ON RC.c_id = R.r_consequencesid WHERE R.r_riskcenterid = :riskcenterid GROUP BY r_consequencesid ORDER BY monetizedrisk DESC",
      "parameters":[
        { "type":"long", "placeholder": ":riskcenterid", "sessionparameter": "siteid" }
      ]
    },
    "chartdataglue":[
      { "type":"array", "placeholder":"#labels", "sqlfield":"c_name" },
      { "type":"array", "placeholder":"#amounts", "sqlfield":"monetizedrisk" }
    ],
    "graphmeta": {
      "title": "Top 10 Consequences"
    },
    "chart": {
      "type":"horizontalBar",
      "data": {
        "labels": "#labels",
        "datasets": [{
          "label": "Total monetised risk: ",
          "data": "#amounts",
          "backgroundColor:":[
            "rgba(255, 99, 132, 0.2)",
            "rgba(54, 162, 235, 0.2)",
            "rgba(255, 206, 86, 0.2)",
            "rgba(75, 192, 192, 0.2)",
            "rgba(153, 102, 255, 0.2)",
            "rgba(255, 159, 64, 0.2)",
            "rgba(255, 159, 64, 0.2)",
            "rgba(255, 159, 64, 0.2)",
            "rgba(255, 159, 64, 0.2)"
          ],
          "borderColor": [
            "rgba(255,99,132,1)",
            "rgba(54, 162, 235, 1)",
            "rgba(255, 206, 86, 1)",
            "rgba(75, 192, 192, 1)",
            "rgba(153, 102, 255, 1)",
            "rgba(255, 159, 64, 1)",
            "rgba(255, 255, 255, 1)",
            "rgba(153, 0, 76, 1)",
            "rgba(0, 102, 0, 1)",
            "rgba(51, 0, 102, 1)"
          ],
          "borderWidth": "1"
        }]
      },
      "options": {
        "responsive": true,
        "title":{
          "display":true,
          "text":"Top 10 Consequences"
        },
        "legend": {
          "display":false
        },
        "tooltips": {
          "mode":"index",
          "intersect": true
        },
        "hover": {
          "mode":"nearest",
          "intersect": true
        },
        "scales": {
          "xAxes": [{
            "display": true,
            "scaleLabel": {
              "display": true,
              "labelString": "Risk",
              "scaleStartValue": 0
            },
            "ticks": {
              "suggestedMin": 0,
              "beginAtZero": true
            }
          }],
          "yAxes": [{
            "display": true,
            "scaleLabel": {
              "display": true,
              "labelString": "Consequences",
              "scaleStartValue": 0
            },
            "ticks": {
              "suggestedMin": 0,
              "beginAtZero":true
            }
          }]
        }
      }
    }
  }
}
```