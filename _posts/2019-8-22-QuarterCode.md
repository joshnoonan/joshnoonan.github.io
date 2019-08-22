---
layout: post
title: Python Code for Scraping and Organizing Quarter Shooting Data
---

In my last article, I talked about the work I did calculating teams' shooting percentage by quarter and the associated z-score for that shooting percentage. In this post, I'm going to go over the code used to retrieve and manipulate this data.

First, we will import the necessary packages and also scrape the data from stats.nba.com. Replace the *** in the code with the particular year that you are looking at.

```python

import requests
import numpy as np
import pandas as pd
import copy
import csv

shot_chart_url = "https://stats.nba.com/stats/shotchartdetail?Period=0&VsConference=&LeagueID=00&LastNGames=0&TeamID=0&PlayerPosition=&Location=&Outcome=&ContextMeasure=FGA&DateFrom=&StartPeriod=&DateTo=&OpponentTeamID=0&ContextFilter=&RangeType=&Season=****-**&AheadBehind=&PlayerID=0&EndRange=&VsDivision=&PointDiff=&RookieYear=&GameSegment=&Month=0&ClutchTime=&StartRange=&EndPeriod=&SeasonType=Regular%20Season&SeasonSegment=&GameID="

headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; U; PPC Mac OS X; en) AppleWebKit/124 (KHTML, like Gecko) Safari/125'}
# Get the webpage containing the data
response = requests.get(shot_chart_url, headers=headers)
# Grab the headers to be used as column headers for our DataFrame
headers = response.json()['resultSets'][0]['headers']
# Grab the shot chart data
shots = response.json()['resultSets'][0]['rowSet']
```

We'll next create a template dictionary 'd' that each team will end up with a copy of; the dictionary 'allD' will hold all of the team dictionaries mapping quarters to shot attempts.

```python
d ={'1' : [], '2': [], '3': [], '4':[]}
allD = {}
```

Now let's fill in these dictionaries.

```python
for row in shots:
    if row[6] not in allD.keys():
        allD[row[6]] = copy.deepcopy(d) #each team corresponds to a copy of template dict 'd'
for row in shots:
    if int(row[7]) < 5: # looking at strictly the first four quarters, this code makes sure to exclude overtime
        allD[row[6]][row[7]].append(row[10])
```

We'll now create a quick function that calculates shooting percentage.
```python
def shootper(l):
    return (l.count('Made Shot') / len(l))
```

Calculate shooting percentages for each quarter for each team.
```python
for team in allD.keys():
    allD[team]['1'] = shootper(allD[team]['1'])
    allD[team]['2'] = shootper(allD[team]['2'])
    allD[team]['3'] = shootper(allD[team]['3'])
    allD[team]['4'] = shootper(allD[team]['4'])
```
Make a list for each quarter of every team's shooting percentage in that respective quarter.
```python
firstQ = [allD[team]['1'] for team in allD.keys()]
secondQ =[allD[team]['2'] for team in allD.keys()]
thirdQ = [allD[team]['3'] for team in allD.keys()]
fourthQ = [allD[team]['4'] for team in allD.keys()]
```

Calculate means and standard deviations for each quarter

```python
firstMean = np.mean(firstQ)
secondMean = np.mean(secondQ)
thirdMean = np.mean(thirdQ)
fourthMean = np.mean(fourthQ)

firstSTD = np.std(firstQ)
secondSTD = np.std(secondQ)
thirdSTD = np.std(thirdQ)
fourthSTD = np.std(fourthQ)
```

Calculate z-score for each team's shooting percentage in a quarter.
```python
teams = [key for key in allD.keys()] #list of teams
#Create dictionaries for each quarter that hold tuple of format
##(shooting percentage, z score)
firstQuarter = {}
for x,y in zip(teams,firstQ):
    firstQuarter[x] = (y, ((y-firstmean)/(firstSTD))

secondQuarter = {}
for x,y in zip(teams,secondQ):
    secondQuarter[x] = (y, ((y-secondmean)/secondSTD))

thirdQuarter = {}
for x,y in zip(teams,thirdQ):
    thirdQuarter[x] = (y, ((y-thirdmean)/thirdSTD))

fourthQuarter = {}
for x,y in zip(teams,fourthQ):
    fourthQuarter[x] = (y, ((y-fourthmean)/fourthSTD))
#put all these dictionaries in a list
quarters = [firstQuarter, secondQuarter, thirdQuarter, fourthQuarter]
```

Create a final list 'qList' to hold all of this data and save as a csv file.
```python
qList = [['Team Name', 'Quarter','Shooting Percentage', 'Z-Score']]

for x,y in zip(quarters,(range(1,4+1))): #range represents quarter
    for team in x.keys():
               qList.append([team, y, x[team][0], x[team][1]]) #format is [name, quarter, shooting percentage, z-score
with open('****teams.csv', 'w') as csvFile:
    writer = csv.writer(csvFile)
    writer.writerows(qList)
```

And there we have it! If you have encounter any errors in the code or have any suggestions on how to clean it up, feel free to reach out!
