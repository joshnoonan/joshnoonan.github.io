---
layout: post
title: Python Code for the Height Visualization
---

In my last post I highlighted some scatterplots that I made using Python that foucsed on modeling the average shot distance and height of players on two different NBA championship teams. In this post, I'm going to include the code that I used to make these visualizations, and take my first crack at creating code blocks in Markdown. 

First we'll import all the necessary packages
```python
import csv
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
```

Then we'll create a function called average(list) that will calculate the average shot distance of a player.
```python
def averageL(l):
    return sum(l) / len(l)
```

Now let's read in the csv file created with data from stats.nba.com of all shots taken by the Chicago Bulls from 1996-98.
```python
with open('9698.csv','r') as f:
        csvf = csv.reader(f)
        result = []
        for row in csvf:
            if row[1].isdigit(): result.append(row) 
```

We'll create a dictionary 'd', that maps a player name to the distance of each of their shots as we read the csv file.
```python
d ={}
for row in result: #map player names to all shot distances
    if row[4] not in d.keys(): #if player not already in dictionary
        d[row[4]] = (float(row[16]),) #row[16] holds the shot distance
    else:
        d[row[4]] += (float(row[16]),)
for key in d.keys():
    d[key] = (averageL(d[key]),) #compute average shot distance
```

Now we'll add in the heights of each player (data taken from the NBA website)
```python
# add heights
d['Luc Longley'] += (86,)
d['Jason Caffey'] += (80,)
d['Ron Harper'] += (78,)
d['Michael Jordan'] += (78,)
d['Dennis Rodman'] += (79,)
d['Scottie Pippen'] += (80,)
d['Toni Kukoc'] += (83,)
d['Bison Dele'] += (83,)
d['Jud Buechler'] += (78,)
d['Randy Brown'] += (75,)
d['Steve Kerr'] += (75,)
d['Robert Parish'] += (84,)
```

Create the x and y variables to plot.
```python
x = [d[key][1] for key in d.keys()] #x-var = height
y = [d[key][0] for key in d.keys()] #y-var = avg shot dist
keys = [key for key in d.keys()]
```

Plot using matplotlib.
```python
plt.scatter(x, y, c= 'r', zorder = 10)
plt.xlabel('Height (in inches)')
plt.xlim(left = 72, right = 88)
plt.ylim(bottom = 0, top = 25)
plt.ylabel('Average shot distance (in feet)')
plt.title('Height vs Avg. Shot Distance for Chicago Bulls players (1996-1998)')
```

Annotate each point with the player name.
```python
i = 0
for x,y in zip(x,y): #annotate each point with respective player
   plt.text(x * (1 + 0.005), y * (1 + 0.005) , keys[i], fontsize=6.9)
   i += 1
```

Add the finishing touches including the light grey shading in the upper right hand corner that is used to highlight differences with the shot chart of the Warriors.
```python
plt.text(70,-2.3,'Data Source: stats.nba.com', fontsize=7)
plt.axhspan(15,25, .38,1,  facecolor='lightgrey')
plt.savefig('CHIscatter.pdf')
plt.show()
```
