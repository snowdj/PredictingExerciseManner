Practical Machine Learning/ Prediction Assignment
========================================================


Background
==============

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement  a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. 

One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this data set, the participants were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 


In this project, the goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants toto predict the manner in which praticipants did the exercise.

The dependent variable or response is the "classe" variable in the training set.  


Data
==============


Download and load the data.

```r
#download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", destfile = "./pml-training.csv")
#download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", destfile = "./pml-testing.csv")

setwd("E:\\Dropbox\\Github\\Rcourses\\08_PracticalMachineLearning\\project")

trainingOrg = read.csv("pml-training.csv", na.strings=c("", "NA", "NULL"))
# data.train =  read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", na.strings=c("", "NA", "NULL"))

testingOrg = read.csv("pml-testing.csv", na.strings=c("", "NA", "NULL"))
dim(trainingOrg)
```

```
## [1] 19622   160
```

```r
dim(testingOrg)
```

```
## [1]  20 160
```


### Pre-screening the data

There are several approaches for reducing the number of predictors. 


- Remove variables that we believe have too many NA values.



```r
training.dena <- trainingOrg[ , colSums(is.na(trainingOrg)) == 0]
#head(training1)
#training3 <- training.decor[ rowSums(is.na(training.decor)) == 0, ]
dim(training.dena)
```

```
## [1] 19622    60
```

- Remove unrelevant variables
There are some unrelevant variables that can be removed as they are unlikely to be related to dependent variable.


```r
remove = c('X', 'user_name', 'raw_timestamp_part_1', 'raw_timestamp_part_2', 'cvtd_timestamp', 'new_window', 'num_window')
training.dere <- training.dena[, -which(names(training.dena) %in% remove)]
dim(training.dere)
```

```
## [1] 19622    53
```


- Check the variables that have extremely low variance (this method is useful nearZeroVar() )


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
# only numeric variabls can be evaluated in this way.

zeroVar= nearZeroVar(training.dere[sapply(training.dere, is.numeric)], saveMetrics = TRUE)
training.nonzerovar = training.dere[,zeroVar[, 'nzv']==0]
dim(training.nonzerovar)
```

```
## [1] 19622    53
```

- Remove highly correlated variables 90% (using for example findCorrelation() )


```r
# only numeric variabls can be evaluated in this way.
corrMatrix <- cor(na.omit(training.nonzerovar[sapply(training.nonzerovar, is.numeric)]))
dim(corrMatrix)
```

```
## [1] 52 52
```

```r
# there are 52 variables.
corrDF <- expand.grid(row = 1:52, col = 1:52)
corrDF$correlation <- as.vector(corrMatrix)
levelplot(correlation ~ row+ col, corrDF)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

We are going to remove those variable which have high correlation.



```r
removecor = findCorrelation(corrMatrix, cutoff = .90, verbose = TRUE)
```




```r
training.decor = training.nonzerovar[,-removecor]
dim(training.decor)
```

```
## [1] 19622    46
```


We get 19622 samples and 46 variables.


### Split data to training and testing for cross validation.


```r
inTrain <- createDataPartition(y=training.decor$classe, p=0.7, list=FALSE)
training <- training.decor[inTrain,]; testing <- training.decor[-inTrain,]
dim(training);dim(testing)
```

```
## [1] 13737    46
```

```
## [1] 5885   46
```

We got 13737 samples and 46 variables for training, 5885 samples and 46 variables for testing.



## Analysis

### Regression Tree 
Now we fit a tree to these data, and summarize and plot it. First, we use the 'tree' package. It is much faster than 'caret' package.


```r
library(tree)
set.seed(12345)
tree.training=tree(classe~.,data=training)
summary(tree.training)
```

```
## 
## Classification tree:
## tree(formula = classe ~ ., data = training)
## Variables actually used in tree construction:
##  [1] "pitch_forearm"     "magnet_belt_y"     "accel_forearm_z"  
##  [4] "magnet_dumbbell_y" "roll_forearm"      "magnet_dumbbell_z"
##  [7] "pitch_belt"        "yaw_belt"          "accel_dumbbell_y" 
## [10] "accel_forearm_x"   "accel_dumbbell_z"  "gyros_belt_z"     
## Number of terminal nodes:  24 
## Residual mean deviance:  1.515 = 20770 / 13710 
## Misclassification error rate: 0.2889 = 3969 / 13737
```

```r
plot(tree.training)
text(tree.training,pretty=0, cex =.8)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 
This is a bushy tree, and we are going to prune it.

For a detailed summary of the tree, print it:

```r
#tree.training
```

### Rpart form Caret, very slow.


```r
library(caret)
modFit <- train(classe ~ .,method="rpart",data=training)
print(modFit$finalModel)
```

```
## n= 13737 
## 
## node), split, n, loss, yval, (yprob)
##       * denotes terminal node
## 
##  1) root 13737 9831 A (0.28 0.19 0.17 0.16 0.18)  
##    2) pitch_forearm< -33.95 1116    6 A (0.99 0.0054 0 0 0) *
##    3) pitch_forearm>=-33.95 12621 9825 A (0.22 0.21 0.19 0.18 0.2)  
##      6) magnet_belt_y>=555.5 11593 8799 A (0.24 0.23 0.21 0.18 0.15)  
##       12) magnet_dumbbell_y< 426.5 9533 6820 A (0.28 0.18 0.24 0.17 0.12)  
##         24) roll_forearm< 120.5 5902 3478 A (0.41 0.17 0.18 0.14 0.089) *
##         25) roll_forearm>=120.5 3631 2422 C (0.08 0.19 0.33 0.22 0.18)  
##           50) accel_forearm_x>=-106.5 2579 1621 C (0.088 0.23 0.37 0.093 0.22) *
##           51) accel_forearm_x< -106.5 1052  498 D (0.06 0.086 0.24 0.53 0.089) *
##       13) magnet_dumbbell_y>=426.5 2060 1112 B (0.039 0.46 0.049 0.21 0.24) *
##      7) magnet_belt_y< 555.5 1028  189 E (0.0019 0.0039 0.00097 0.18 0.82) *
```

### Prettier plots


```r
library(rattle)
```

```
## Rattle: A free graphical interface for data mining with R.
## XXXX 3.0.2 r169 Copyright (c) 2006-2013 Togaware Pty Ltd.
## Type 'rattle()' to shake, rattle, and roll your data.
```

```r
fancyRpartPlot(modFit$finalModel)
```

```
## Loading required package: rpart.plot
## Loading required package: rpart
## Loading required package: RColorBrewer
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 

The result from 'caret' 'rpart' package is close to 'tree' package.


### Cross Validation

We are going to check the performance of the tree on the testing data by cross validation.


```r
tree.pred=predict(tree.training,testing,type="class")
predMatrix = with(testing,table(tree.pred,classe))
sum(diag(predMatrix))/sum(as.vector(predMatrix)) # error rate
```

```
## [1] 0.7158879
```

The 0.70 is not very accurate.


```r
tree.pred=predict(modFit,testing)
predMatrix = with(testing,table(tree.pred,classe))
sum(diag(predMatrix))/sum(as.vector(predMatrix)) # error rate
```

```
## [1] 0.4934579
```
The 0.50 from 'caret' package is much lower than the result from 'tree' package.


### Pruning tree
This tree was grown to full depth, and might be too variable. We now use Cross Validation to prune it.

```r
cv.training=cv.tree(tree.training,FUN=prune.misclass)
cv.training
```

```
## $size
##  [1] 24 22 21 20 19 18 17 16 13  7  5  1
## 
## $dev
##  [1] 4531 4649 4658 4667 5086 5131 5230 5324 6659 6727 7254 9831
## 
## $k
##  [1]     -Inf  87.0000  93.0000  99.0000 137.0000 143.0000 147.0000
##  [8] 164.0000 191.3333 199.0000 267.5000 650.5000
## 
## $method
## [1] "misclass"
## 
## attr(,"class")
## [1] "prune"         "tree.sequence"
```

```r
plot(cv.training)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 

It shows that when the size of the tree goes down, the deviance goes up. It means the 21 is a good size (i.e. number of terminal nodes)  for this tree. We do not need to prune it.

Suppose we prune it at size of nodes at 18.


```r
prune.training=prune.misclass(tree.training,best=18)
#plot(prune.training);text(prune.training,pretty=0,cex =.8 )
```




Now lets evaluate this pruned tree on the test data.





```r
tree.pred=predict(prune.training,testing,type="class")
predMatrix = with(testing,table(tree.pred,classe))
sum(diag(predMatrix))/sum(as.vector(predMatrix)) # error rate
```

```
## [1] 0.6677995
```

0.66 is a little less than 0.70, so pruning did not hurt us with repect to misclassification errors, and gave us a simpler tree.
We use less predictors to get almost the same result.
By pruning, we got a shallower tree, which is easier to interpret.


The single tree is not good enough, so we are going to use bootstrap to improve the accuracy. We are going to try random forests.

Random Forests 
============================

These methods use trees as building blocks to build more complex models.

Random Forests
--------------
Random forests build lots of bushy trees, and then average them to reduce the variance.


```r
require(randomForest)
```

```
## Loading required package: randomForest
## randomForest 4.6-7
## Type rfNews() to see new features/changes/bug fixes.
```

```r
set.seed(12345)
```

Lets fit a random forest and see how well it performs.


```r
rf.training=randomForest(classe~.,data=training,ntree=100, importance=TRUE)
rf.training
```

```
## 
## Call:
##  randomForest(formula = classe ~ ., data = training, ntree = 100,      importance = TRUE) 
##                Type of random forest: classification
##                      Number of trees: 100
## No. of variables tried at each split: 6
## 
##         OOB estimate of  error rate: 0.72%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3899    4    0    1    2 0.001792115
## B   23 2622   13    0    0 0.013544018
## C    0   13 2379    4    0 0.007095159
## D    0    0   27 2224    1 0.012433393
## E    0    1    2    8 2514 0.004356436
```

```r
#plot(rf.training, log="y")
varImpPlot(rf.training,)
```

![plot of chunk unnamed-chunk-18](figure/unnamed-chunk-18.png) 

```r
#rf.training1=randomForest(classe~., data=training, proximity=TRUE )

#DSplot(rf.training1, training$classe)
```

we can see which variables have higher impact on the prediction.


## Out-of Sample Accuracy

Our Random Forest model shows OOB estimate of error rate: 0.72% for the training data. Now we will predict it for out-of sample accuracy.

Now lets evaluate this tree on the test data.



```r
tree.pred=predict(rf.training,testing,type="class")
predMatrix = with(testing,table(tree.pred,classe))
sum(diag(predMatrix))/sum(as.vector(predMatrix)) # error rate
```

```
## [1] 0.9947324
```


0.99 means we got a very accurate estimate. 

No. of variables tried at each split: 6. It means every time we only randomly use 6 predictors to grow the tree. Since p = 43,  we can have it from 1 to 43, but it seems 6 is enough to get the good result.

Conclusion
================




Now we can predict the testing data from the website.


```r
answers <- predict(rf.training, testingOrg)
answers
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```




Those answers are going to submit to website for grading.  It shows that this random forest model did a good job.
