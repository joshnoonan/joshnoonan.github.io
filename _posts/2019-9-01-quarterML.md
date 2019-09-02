---
layout: post
title: Predicting Playoff Teams Using Quarter Shooting Data
---

In this post, I'll share the Python code and process that I went through in applying machine learning techniques to the quarter shooting data. I applied a k-nearest neighbors algorithm to the data and achieved 66% accuracy working with a single data parameter.

As always, we need to import the packages that we are going to use.
```python
import matplotlib.pyplot as plt
import pandas as pd
import pylab as pl
import numpy as np
%matplotlib inline
```

Then we'll read in our data.
```python
df = pd.read_csv("completeshooting9619.csv")

# take a look at the dataset
df.head()
```
We need to add a column to our dataframe with 0 = no playoff berth and 1 = playoff berth.
```python
df['Playoff'] = [1 if x != 0 else 0 for x in df['PlayoffPoints']]
qdf = df[['Team Name','ZScore','PlayoffPoints','Playoff']] #select the features we want to look at
```
Make training and test data sets.
```python
from sklearn.model_selection import train_test_split
X = qdf['ZScore'].values.reshape(-1, 1)
y = qdf['Playoff'].values
X_train, X_test, y_train, y_test = train_test_split( X2, y2, test_size=0.2, random_state=4)
``` 
We'll now construct our model, first setting k equal to a completely arbitrary value of 10.
```python
from sklearn.neighbors import KNeighborsClassifier
k = 18
#Train Model and Predict  
neigh = KNeighborsClassifier(n_neighbors = k).fit(X_train,y_train)
yhat = neigh2.predict(X_test)
```

We'll now measure the accuracy of the model for k = 10.
```python
from sklearn import metrics
print("Train set Accuracy: ", metrics.accuracy_score(y2_train, neigh.predict(X2_train)))
print("Test set Accuracy: ", metrics.accuracy_score(y2_test, yhat2))
```

This gives us a training set accuracy of 59.1% and a test set accuracy of 63.4%. Let's now run the k-nearest neighbors algorithm for every k less than or equal to 30.
```python
Ks = 30
mean_acc = np.zeros((Ks-1))
std_acc = np.zeros((Ks-1))
ConfustionMx = [];
for n in range(1,Ks):
    
    #Train Model and Predict  
    neigh = KNeighborsClassifier(n_neighbors = n).fit(X_train,y_train)
    yhat =neigh.predict(X_test)
    mean_acc[n-1] = metrics.accuracy_score(y_test, yhat)

    
    std_acc[n-1]=np.std(yhat2==y2_test)/np.sqrt(yhat2.shape[0])

mean_acc

print( "The best accuracy was with", mean_acc.max(), "with k =", mean_acc.argmax()+1) 
```
The k-value that produced the greatest accuracy was k = 18 with an accuracy of 65.6%. Let's now create a plot that visualizes the accuracy for each k in the range from 1 to 30.
```python
plt.plot(range(1,Ks),mean_acc2,'g')
plt.fill_between(range(1,Ks),mean_acc2 - 1 * std_acc2,mean_acc2 + 1 * std_acc2, alpha=0.10)
plt.legend(('Accuracy ', '+/- 3xstd'))
plt.ylabel('Accuracy ')
plt.xlabel('Number of Neighbors (K)')
plt.tight_layout()
plt.show()
```
![Here's our plot](https://raw.githubusercontent.com/joshnoonan/joshnoonan.github.io/master/images/quarterK.png) 
