---
description: >-
  A simple way to implement charts with your AdminJS instance is to use Recharts
  library and Admin's API.
---

# Charts

In order to provide custom components on your dashboard you will have to override the dashboard entirely. The [dashboard customization](../ui-customization/dashboard-customization.md) article will come in handy.

In your dashboard handler, you will need to fetch and format the required data for charts. For purposes of this guide, we will be using [Recharts](https://recharts.org/), which requires data to be passed in an array of objects.

{% code title="Recharts data variable" %}
```typescript
[
    {
        name: xAxisVariable0,
        value: yAxisVariable0
    },
    {
        name: xAxisVariable1,
        value: yAxisVariable1
    },
    // ...
    {
        name: xAxisVariableN,
        value: yAxisVariableN
    },
]
```
{% endcode %}

Let's assume our database contains a table called movies, which contains three columns - 'title', 'year', and 'score'.

{% code title="dashboard.handler.ts" %}
```tsx
import { Filter } from 'adminjs'

export const dashboardHandler = async (request, response, context) => {
  // finding resource called movies
  const resource = context._admin.findResource('movies')
  // creating new filter, so that we can see only movies released in 2020
  const filter = new Filter({}, resource)
  // finding all records that match provided filter
  const resourceData = await resource.find(filter, { sort: { sortBy: 'year', direction: 'desc' } }, context)
  
  const data = resourceData.map((item) => item.toJSON(context.currentAdmin))
  
  return data
}
```
{% endcode %}

However, currently, the data variable is an array that contains all the records in the following manner.

{% code title="data variable" %}
```typescript
[
{
    params: {
        id: 2104,
        title: 'Arrival',
        year: 2018,
        score: 84
    },
    baseError: null,
    // ...
},
{
    params: {
        id: 2226,
        title: 'Harry Potter and the Goblet of Fire',
        year: 2018,
        score: 96
    },
    baseError: null,
    // ...
},
// ...
]
```
{% endcode %}

We will need to parse to Recharts data format.

{% code title="dashboard.handler.ts" %}
```typescript
// ...
  const years = Array.from(new Set(data?.map((item) => item.params.year))) // Set leaves only unique values, but we need an Array
  const chartdata = years.map(year => { // for every year that we've got
    const scoreArr = data?.filter(filterItem => filterItem.params.year === year) // find movies from a certain year
                          .map(mapItem => mapItem.params.score) // create an array of all the scores from that year
    return (
      {
        name: year,
        score: scoreArr.reduce((a, b) => a + b, 0) / scoreArr.length // create average valuefrom the score array
      })
  })
  return chartdata
```
{% endcode %}

The next step is creating a chart component we will later on put into our dashboard.

{% code title="linechart.component.tsx" %}
```typescript
import type React from 'react'

import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend } from 'recharts'

export const LineChartComponent: React.FC = ({ data }) => {
  return (
        <LineChart
          width={500}
          height={300}
          data={data}
          margin={{
            top: 5,
            right: 30,
            left: 20,
            bottom: 5
          }}
        >
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis dataKey="name" />
          <YAxis />
          <Tooltip />
          <Legend />
          <Line type="monotone" dataKey="score" stroke="#8884d8" activeDot={{ r: 8 }} />
        </LineChart>
  )
}

```
{% endcode %}

The last part is adding the line chart component to our dashboard.

{% code title="dashboard.tsx" %}
```tsx
import { ApiClient } from 'adminjs'
import React, { useState, useEffect } from 'react'

import { LineChartComponent } from './linechart.component.js'

const Dashboard: React.FC = () => {
  const [data, setData] = useState(null)
  const api = new ApiClient()

  useEffect(() => {
    api.getDashboard()
      .then((response) => {
        setData(response.data)
      })
      .catch((error) => {
        // Handle errors here
      })
  }, [])
  return (
    <LineChartComponent data={data} />
  )
}

export default Dashboard
```
{% endcode %}
