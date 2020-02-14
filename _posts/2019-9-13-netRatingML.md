---
layout: post
title: Predicting Playoff Teams Using Net Rating
---

As promised in my last post, I'll be using machine learning to predict playoff probabilities for teams in today's post. It's still the same general concept, but its product is a little more informative! As a quick refresher, net rating is a measure of a team's point differential per 100 possessions. It is calculated by subtracting a team's defensive rating (points allowed per 100 possessions) from their offensive rating (points scored per 100 possessions). Net rating itself is a fairly strong predictor of a team's playoff chances, as at a net rating of roughly 3.75 and higher, no team has ever failed to make the playoffs. 

In this post, I've used some machine learning techniques with the sci-kit learn library to produce some fun results.

I feel like a broken record saying this, but we'll start out with importing the necessary packages.

```python
import matplotlib.pyplot as plt
import pandas as pd
import pylab as pl
import numpy as np
```

We'll now read in the csv file holding teams' net ratings, quarter shooting percentages, z-scores, and whether or not they made the playoffs. This file holds all this information for NBA teams from the 1996-97 season to the 2018-19 season. We'll be training our model using the data up to and including the 2017-18 season, and then making predictions on the 2018-19 data.

```python
df = pd.read_csv("everything.csv")
df['Playoff'] = [1 if x != 0 else 0 for x in df['PlayoffPoints']]
df.head(10)
```

Let's separate the 2018-19 season into a separate dataframe that will be used as the test data and also trim the 2018-19 data from the master dataframe.

```python
currentszn = df.head(120)
df = df.iloc[120:]
df.head()
```

Time to get underway with the SVM!

```python
X = df[['Zscore','NRtg']]
y = df['Playoff'].values
X_test = currentszn[['Zscore','NRtg']]
```

Let's set up the model and then train it using the data from 1996-2018
```python
from sklearn import svm
probmodel = svm.SVC(probability=True)
probmodel.fit(X,y)
```
Now let's use the SVM to predict the playoff probabilities of teams in the 2018-19 NBA season.

```python
probmodel.predict_proba(X_test)
probmodelpredict = probmodel.predict_proba(X_test)
probmodelpredict
```

Let's add these playoff probabilites to the dataframe representing the 2018-2019 season.
```python
currentszn["PlayoffProb"] = probmodelpredict[:,1] #[:,1] isolates probability of making playoffs
currentszn.head()
```

Average every four values in the probability array in order to compute a single playoff probability for each team.

```python
playoffprobs = np.mean(probmodelpredict[:,1].reshape(-1, 4), axis=1)
playoffprobs
```

Let's now make a new dataframe holding just the team names and their playoff probabilities.
```python 
predict = pd.DataFrame(columns = [["Team", "PlayoffProb"]])
predict["Team"] = currentszn["Team Name"]
predictions = predict.drop_duplicates()
# let's clean it up a bit now
predictions["PlayoffProb"] = playoffprobs * 100
predictions = predictions.reset_index() # reset row index 
predictions
```

Here's our predictions!

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>


|Team   | Playoff Probability  |
|---|---|
|  ATL |  6.374205 |
|  BOS | 89.921918 |
|  BKN | 85.736312 |
|  CHO |  8.575410 |
| CHI  | 6.284226  |
| CLE  | 6.276276  |
|  DAL | 5.737062  |
| DEN  | 90.094917 |
|  DET |  85.711793 |
| GSW  | 90.177350  |
|  HOU | 90.214094  |
|  IND |  90.724214 |
| LAC  |  91.362140 |
|  LAL |  5.006783 |
|  MEM |  6.109226 |
|  MIA | 74.629421  |
|  MIL | 90.187167  |
| MIN  |  4.818284 |
| NOP  | 5.431897  |
|  NYK |   6.202574|
| OKC  |  90.077388 |
|  ORL |  90.537884 |
| PHI  |  88.658685 |
| PHX  | 6.235367  |
|  POR | 89.917895  |
| SAC  | 7.687028  |
|  SAS |  88.751595 |
| TOR  |  90.056788 |
|  UTA |  90.018541 |
|  WAS | 5.951006  |
{: .tablelines}

Let's save our predictions as a csv file.

```python
predictions.to_csv("1819predictions.csv")
```

As we can see, our model was pretty accurate in predicting what teams would make the playoffs. Our model was high on the Orlando Magic, a team that many pundits and analysts left for dead in the earlier half of the season. The model did give the Miami Heat a roughly 75% chance of making the playoffs, and the Heat eventually ended the season on the outside looking in as the 9-seed. I wouldn't consider this a failure by the model as the Heat were in contention for a playoff spot until the literal last day of the regular season. Overall, I'm happy with how the model turned out, and look forward to utilizing SVMs in later projects!

