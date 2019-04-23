
# Philippine Earthquakes EDA

by Prince Javier

We explore earthquake data from USGS from 1980 to April 23, 2019 inside the region bounded by:
<br>Latitude: (4.442, 19.692)
<br>Longitude: (115.664, 129.727)

*This is a work in progress.*

<details>
<p>
  
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.plotly as py
import plotly.graph_objs as go
import datetime as dt

import warnings
warnings.filterwarnings("ignore")
```

```python
eq = pd.read_csv("data_other/earthquakes_1980_onwards.csv")
```


```python
eq["depth"] = eq["depth"] * -1
```


```python
eq["time"] = pd.to_datetime(eq["time"])
```


```python
## convert to GMT + 8
eq["date"] = eq["time"] + dt.timedelta(hours=8)
```


```python
# text
eq["text"] = [f"mag: {mag}<br>depth: {depth}<br>date: {date.date()}" for mag,
              depth, date in zip(eq.mag, eq.depth, eq.date)]
```
</details>

## Spatial Distribution of Earthquakes in Ph
  
<details>
  
```python
# Create a trace
data = go.Scattergeo(
    lon=eq.longitude,
    lat=eq.latitude,
    text=eq.text,
    marker=go.scattergeo.Marker(
        color=eq.depth,
        size=2 ** eq.mag/10,
        colorscale="Jet",
        colorbar=dict(thickness=10, title="Depth")),
)

layout = go.Layout(
    title=go.layout.Title(
        text='Earthquake in the Ph from 1980 to April 23, 2019'),

    width=700,
    height=800,
    showlegend=False,

    geo=go.layout.Geo(
        resolution=50,
        scope='asia',
        showframe=False,
        showcoastlines=True,
        showland=True,
        landcolor="rgb(229, 229, 229)",
        countrycolor="rgb(255, 255, 255)",
        coastlinecolor="rgb(255, 255, 255)",
        projection=go.layout.geo.Projection(
            type='mercator'
        ),
        lonaxis=go.layout.geo.Lonaxis(
            range=[116.0, 129.0]
        ),
        lataxis=go.layout.geo.Lataxis(
            range=[5, 20]
        ),
        domain=go.layout.geo.Domain(
            x=[0, 1],
            y=[0, 1]
        )
    ),
    legend=go.layout.Legend(
        traceorder='reversed'
    )
)

data = [data]
fig = go.Figure(layout=layout, data=data)
py.iplot(fig, filename='eq_geoscatter')
```

</details>

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~pjavier/16.embed" height="800px" width="700px"></iframe>


## Distribution of Magnitudes


<details>

```python
mags = eq.mag.value_counts()
x = mags.keys()
inds = np.argsort(x)
x = x[inds]
y = mags.values[inds]

# Create a trace
data = go.Scatter(
    x=x,
    y=y
)

# Edit the layout
layout = dict(title='Histogram of Earthquake Magnitudes Around Ph',
              xaxis=dict(title='Magnitude'),
              yaxis=dict(title='Counts'),
              width=800,
              height=400
              )

data = [data]

fig = dict(data=data, layout=layout)
py.iplot(fig, filename='eq_distribution_mag')
```
</details>



<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~pjavier/18.embed" height="400px" width="800px"></iframe>



## Distribution of Time between Consecutive Earthquakes


<details>

```python
# @hidden_cell
from collections import Counter
```


```python
eq = eq.sort_values("time")
```


```python
data = []

eq_ranges = [(2.5, 5), (5, 9)]

for i in range(len(eq_ranges)):
    min_, max_ = eq_ranges[i]
    eq_filtered = eq[np.logical_and(eq.mag >= min_, eq.mag <= max_)]

    eq_filtered["time_gap"] = [0] + [t2 - t1 for t1,
                                     t2 in zip(eq_filtered.time[:-1], eq_filtered.time[1:])]

    # get hour gaps
    hour_gaps = [0] + [i.days * 24 for i in eq_filtered.time_gap.values[1:]]
    hour_gaps = Counter(hour_gaps)

    x = np.array(list(hour_gaps.keys()))
    inds = np.argsort(x)
    x = x[inds]
    y = np.array(list(hour_gaps.values()))[inds]
    y = np.log(y)

    # Create a trace
    trace = go.Scatter(
        name=str(eq_ranges[i]),
        x=x,
        y=y
    )

    data.append(trace)

# Edit the layout
layout = dict(title='Distribution of Gaps Between Earthquakes for Different Magnitude Ranges',
              xaxis=dict(title='Hours gap between consecutive earthquakes',
                         range=[0, 1200]),
              yaxis=dict(title='Natural Logarithm of Frequency'),
              width=800,
              height=400
              )


fig = dict(data=data, layout=layout)
py.iplot(fig, filename='eq_distribution_time')
```

</details>


<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~pjavier/20.embed" height="400px" width="800px"></iframe>
