---
description: >-
  A simple way to implement charts with your AdminJS instance is use Recharts
  library and Admin's API.
---

# Charts

In order to provide custom components on your dashboard you will have to override the dashboard entirely. The [dashboard customization](../ui-customization/dashboard-customization.md) article will come in handy.

Once the dashboard has been overwritten we are going to need, the following:

* Axios to send requests to and process Admin's API responses
* An example Recharts chart
* A class that extends React.Component, that will take care of the axios calls, generating the charts and rendering them.

```tsx
// dashboard.component.tsx
import axios from 'axios';
import React from 'react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';

export default class LineChart extends React.Component {
    state = {
        items: [],
    };
    
    componentDidMount() {
        axios.get('apiCallString').then((response) => { // reference the documentation for the API
            const items = response.data.records;
            this.setState({ items })
        }
        )
    };
    
    generateChart() {
    const data = this.state.items.map((record) => {
      return ( // both id and value will have to reference your database table or collection
        { xAxisVariable: record.params.id
          yAxisVariable: record.params.value
        }
      )
    });
        return (
          <ResponsiveContainer width="100%" height="100%">
            <LineChart
              width={500}
              height={300}
              data={data}
              margin={{
                top: 5,
                right: 30,
                left: 20,
                bottom: 5,
              }}
            >
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="xAxisVariable" />
              <YAxis />
              <Tooltip />
              <Legend />
              <Line type="monotone" dataKey="yAxisVariable" stroke="#8884d8" activeDot={{ r: 8 }} />
            </LineChart>
          </ResponsiveContainer>
        )
    };
    
    render() {
      return (
        <>{this.generateChart()}</>
      )  
    };
};
```

The next step is putting the chart inside your custom dashboard.

```tsx
// dashboard.tsx
import React from 'react';
import LineChart from './dashboard.component';

const Dashboard: React.FC = () => {
  return(
    <><LineChart /></>
  )
};

export default Dashboard;
```
