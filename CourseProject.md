# Practical Machine Learning




## Introduction
#### Background
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: [http://groupware.les.inf.puc-rio.br/har] (see the section on the Weight Lifting Exercise Dataset).

#### Data

The training data for this project are available here:

[https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv]

The test data are available here:

[https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv]

The data for this project come from this source: [http://groupware.les.inf.puc-rio.br/har]. If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment.

***

## 1. Loading the Required Libraries and Datasets
Loading the required libraries and extracting the training and testing data from their respective sources.  
Blank elements in the dataset were transformed to "NA".

```r
library(caret)
library(randomForest)
```

```r
training <- read.csv("pml-training.csv",na.strings=c("", "NA"))
testing <- read.csv("pml-testing.csv", na.strings=c("", "NA"))
```

***

## 2. Data Pre-processing and Cleaning
Two transformations were used to clean the data.  
For the first transformation, columns in the dataset which contained 50% or more "NA" elements were removed from both the training and testing data sets.  
For the second transformation, column names containing the words "belt", "arm", "dumbbell" and the predictor variable (classe) were subsetted from the training and testing data sets and then stored into appropriate named variables (modV2_training & modV2_testing).

```r
mod_training <- training
NA_remover <- vector()
for (i in 1:length(training)) {
  if (sum(is.na(training[,i])) / nrow(training) >= 0.5) {
    NA_remover <- c(NA_remover,i)
  }
    
}
mod_training <- training[,-NA_remover]
mod_testing <- testing[,-NA_remover]

selector <- c(grep("belt", colnames(mod_training)),
              grep("arm", colnames(mod_training)),
              grep("dumbbell", colnames(mod_training)),
              ncol(mod_training))

modV2_training <- mod_training[,selector]
modV2_testing <- mod_testing[,selector]
```

***

## 3. Partition the Training Set
The cleaned training data set is partitioned into 2 sets, one for training the algorithm (train_set) and the other for cross-validation (test_set).  
train_set contains 60% of the rows from the cleaned training data.  
test_set contains 40% of the rows from the cleaned training data.  
The seed is set to ensure reproducibility.  

```r
set.seed(123)
inTrain <- createDataPartition(y=modV2_training$classe, p=0.6, list=FALSE)
train_set <- modV2_training[inTrain,]
test_set <- modV2_training[-inTrain,]
```

***

## 4. Predict using Random Forest
The random forest algorithm is used to train the model and a confusion matrix is used to validate the model with the cross-validation data set.

```r
modFit_RF <- randomForest(classe ~ ., data=train_set)
prediction_RF <- predict(modFit_RF, test_set, type = "class")
confusionMatrix(prediction_RF, test_set$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2230   10    0    0    0
##          B    2 1505    9    0    0
##          C    0    3 1355   11    3
##          D    0    0    4 1275    3
##          E    0    0    0    0 1436
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9943          
##                  95% CI : (0.9923, 0.9958)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9927          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9991   0.9914   0.9905   0.9914   0.9958
## Specificity            0.9982   0.9983   0.9974   0.9989   1.0000
## Pos Pred Value         0.9955   0.9927   0.9876   0.9945   1.0000
## Neg Pred Value         0.9996   0.9979   0.9980   0.9983   0.9991
## Prevalence             0.2845   0.1935   0.1744   0.1639   0.1838
## Detection Rate         0.2842   0.1918   0.1727   0.1625   0.1830
## Detection Prevalence   0.2855   0.1932   0.1749   0.1634   0.1830
## Balanced Accuracy      0.9987   0.9948   0.9939   0.9952   0.9979
```
This model appears to have a very high accurary of more than 99%. Hence, we will use this model to predict the classes in the testing set.  
The expected out of sample error is calculated as: 1 - accuracy = 0.0057  

***

## 5. Predict Classes in the Testing Set
The random forest model is used to predict the classes in the testing set and the results are printed below.

```r
prediction_test <- predict(modFit_RF, modV2_testing, type = "class")
prediction_test
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```

***
