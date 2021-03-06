Practical Machine Learning Course Project
========================================================

## Introduction

This is the course project for the Coursera Class: Practical Machine learning. In this project we are going to train a model to predict how well the participants did barbell lifts correctly.

### Brief description of the project from the class

_Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset)._

## Loading and cleaning the data

1. Loading the data  
1.1 We transformed excel error strings () in NA values  
1.2 We specified the predictor values as numeric  
2. We removed the following columns as they don't make sense as to build the model (They can't be used as characteristics of a new data):  
2.1 Line Number  
2.1 User Name  
2.1 Timestamp columns  
2.1 new_window and num_window columns (as they just identify the data)  
3. We removed the predictors that has NA values as they don't contribute for the model  


```r
# Loading the train data and changing excel division error messages to NA values
trainData <- read.csv("data/pml-training.csv", na.strings=c("#DIV/0!",NA))

# For better performance, extract the classe vector
classes <- trainData$classe

# Creating a set of choosen features
# Removing features that has NA values
selectedFeatures <- colnames(trainData)[colSums(is.na(trainData)) == 0]
# Removing some features that does not make sense as predictors
selectedFeatures <- selectedFeatures[-(1:7)]
# Removing the classe feature
selectedFeatures <- selectedFeatures[c(-length(selectedFeatures))]
# Applying this to the training dataset
trainData <- trainData[,selectedFeatures]
# Guarantee that the columns are numeric
for (column in colnames(trainData)[2:ncol(trainData)-1]) {
    trainData[,column] <- as.numeric(as.character(trainData[,column]))
}

# Loading the test dataset and applying the same transformations
testData <- read.csv("data/pml-testing.csv", na.strings=c("#DIV/0!",NA))
testData <- testData[,selectedFeatures]
# Guarantee that the columns are numeric
for (column in colnames(testData)[2:ncol(testData)-1]) {
    testData[,column] <- as.numeric(as.character(testData[,column]))
}
```

# The Random forests model  

We trained our model using the randomForests algorithm. We also did a 10-fold cross validation. 


```r
options(warn=-1)
library(caret)
library(randomForest)

# Makes it reproducible
set.seed(3333)
# Creates the model
resultModel <- train(trainData,classes, tuneGrid=data.frame(mtry=3), method="rf", ntree = 50, trControl = trainControl(method = "cv", number = 10))
resultAccuracy <- format(resultModel$results$Accuracy, digits = 2)
```

The accuracy in the training dataset (in sample error rate) was almost 100%: 1

# Conclusions

As we used the cross-validation to measure our accuracy, we expect the out of sample error to be approximated the same.

# Predicting the test data points


```r
testResult <- as.character(predict(resultModel, testData))
```

```
## Loading required package: randomForest
## randomForest 4.6-12
## Type rfNews() to see new features/changes/bug fixes.
```

```r
# write prediction files
pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("./testResults/test_", i, ".txt")
    write.table(x[i], file = filename, quote = FALSE, row.names = FALSE, col.names = FALSE)
  }
}
pml_write_files(testResult)
```
