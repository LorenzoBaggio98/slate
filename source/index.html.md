---
title: Angus API Docs

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript : NodeJS

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Angus API! You can use our API to access Angus API endpoints, which can get data on your Implant's machines in our Influx Database.

We use the the package " node-influx " ( the official InfluxDB Client for JS ) to run the queries in the Influx DB.

We have also written other APIs to retrieve the other components's data (pH , water level, etc. ) but these are the main ones.
# Total Electric Consumption

## Get the Electric Consumption of the entire Implant

```javascript
app.get("/influx/sumCurrent", function(req, res){
    influx.query(`select * from sum_ElectricCurrent`)
        .then( result => res.status(200).json(result) )
        .catch( error => res.status(500).json({ error }));
})
```

> The above API returns JSON structured like this:

```json

[
    {
        "time": "2018-10-11T12:48:40.000Z",
        "sum": 5346
    },
    {
        "time": "2018-10-11T12:49:00.000Z",
        "sum": 3604
    },
    {
        "time": "2018-10-11T12:49:20.000Z",
        "sum": 5848
    }
]
```

This endpoint retrieves the sum of the Electric Consumption of all the components of each machine inside the Implant.

The sum is calculated every minute from the Continuous Query, in the Influx Database.

### HTTP Request

`GET http://localhost:5000/influx/sumCurrent`

# Component Electric Consumption
## Get the Electric Consumption of a specific Component

```javascript
app.get("/influx/:section/:machine/:component", function(request, response){

    let machine = request.params.machine;
    let section = request.params.section;
    let component = request.params.component;

    influx
        .query(`SELECT mean("electricCurrent") AS "mean_electricCurrent" FROM "GrowingMobile"."autogen"."Angus" WHERE time > now() - 1h AND "component"='${component}' 
        AND "machine"='${machine}' AND "section" = '${section}' GROUP BY time(1m) FILL(linear)`)
        .then( result => response.status(200).json(result) )
        .catch( error => response.status(500).json({error}));
});
```

> The above API returns JSON structured like this:

```json
[
      {
          "time": "2018-10-13T14:28:00.000Z",
          "mean_electricCurrent": 212
      },
      {
          "time": "2018-10-13T14:29:00.000Z",
          "mean_electricCurrent": 605.6
      }
]
```

This endpoint retrieves the Electric Consumption of the specified component


### HTTP Request

`GET http://localhost:5000/influx/<SECTION>/<MACHINE>/<COMPONENT>`

### URL Parameters

Parameter | Description
--------- | -----------
SECTION | The SECTION of the component (contains the machines) 
MACHINE | The MACHINE of the component (contains multiple components) 
COMPONENT | The requested COMPONENT

