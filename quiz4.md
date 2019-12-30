---
title: "Practical Machine Learning - Course Project Week 4"
author: "Prahlad"
date: "28/12/2019"
output: 
  html_document: 
    fig_caption: yes
    keep_md: yes
---



## Executive Summary
Here the qualitative and quantitative data collected by wearable devices (belt, forarms, arm) for 6 participants has been analyzed for barbell lifts both correctly and incorrectly in 5 different ways.
* A- as specified 
* B- streching Elbows to the front (wrong)
* C- Lift dumbbell halfway (wrong) 
* D- Lower dumbbell halfway (wrong) 
* E- move hips to the front (wrong) 
Here first the quality of execution is analyzed  and provide a feed back on correcting the mistakes. This was a controlled exercise. For more info refer  http://groupware.les.inf.puc-rio.br/har. 

Goal of the Project 
1. predict the manner in which they did the exercise by using  "classe" variable in the training set. 
2. Create a report describing how models were built 
3. Used cross validation
4. Expected out of sample error 
5. Reason for making the choices you did. 
6. Use the prediction model to predict 20 different test cases.

The Model has been built to answer the above questions - Use the data base as input, clean the data, use the algorithms to build Training and Test data and then use the algorithms to do the predictions. Cross Validation is used as method for trainControl function.



```
## Warning: package 'knitr' was built under R version 3.6.1
```

```
## Warning: package 'dplyr' was built under R version 3.6.1
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```
## Warning: package 'ggplot2' was built under R version 3.6.1
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```
## Warning: package 'caret' was built under R version 3.6.1
```

```
## Loading required package: lattice
```

```
## Warning: package 'randomForest' was built under R version 3.6.1
```

```
## randomForest 4.6-14
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```
## The following object is masked from 'package:dplyr':
## 
##     combine
```

```
## Warning: package 'rpart.plot' was built under R version 3.6.1
```

```
## Warning: package 'corrplot' was built under R version 3.6.2
```

```
## corrplot 0.84 loaded
```
## Input Data the files used are available at 
The training data:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

```r
training<- read.csv("pml-training.csv") # Training Data
testing<-read.csv("pml-testing.csv")  # Test Data

##Split the data into Training data set and Test data set.

inTrain<-createDataPartition(training$classe, p=.7, list=FALSE)
TrainSet<-training[inTrain,]
TestSet<-training[-inTrain,]
dim(TrainSet)
```

```
## [1] 13737   160
```

```r
dim(TestSet)
```

```
## [1] 5885  160
```
## NZV - near zero values check and remove

```
## [1] 13737   101
```

```
## [1] 5885  101
```

## Check and remove those data that contains more than 95%  of NA. Filter these records.


```
## [1] 13737    59
```

```
## [1] 5885   59
```

## Remove the column 1 to 5 as these are not related to the model.


```r
TrainSet<- TrainSet[, -(1:5)]
TestSet<- TestSet[, -(1:5)]
dim(TrainSet)
```

```
## [1] 13737    54
```

```r
dim(TestSet)
```

```
## [1] 5885   54
```

## Model Random Forest 


```r
set.seed(2345)
library(e1071)
```

```
## Warning: package 'e1071' was built under R version 3.6.2
```

```r
controlRF<-trainControl(method="cv", number=3, verboseIter=FALSE)
modFitRandForest<- train(classe~., data=TrainSet, method="rf", trControl=controlRF)
modFitRandForest$finalModel
```

```
## 
## Call:
##  randomForest(x = x, y = y, mtry = param$mtry) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 27
## 
##         OOB estimate of  error rate: 0.23%
## Confusion matrix:
##      A    B    C    D    E  class.error
## A 3904    1    0    0    1 0.0005120328
## B    4 2650    3    1    0 0.0030097818
## C    0    5 2391    0    0 0.0020868114
## D    0    0    9 2242    1 0.0044404973
## E    0    1    0    5 2519 0.0023762376
```

## Predict Random Forest Model 


```r
predRandForest<- predict(modFitRandForest, newdata=TestSet)
confMatRandForest<-confusionMatrix(predRandForest, TestSet$classe)
confMatRandForest
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1674    6    0    0    0
##          B    0 1133    3    0    0
##          C    0    0 1023    1    0
##          D    0    0    0  963    3
##          E    0    0    0    0 1079
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9978          
##                  95% CI : (0.9962, 0.9988)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9972          
##                                           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   0.9947   0.9971   0.9990   0.9972
## Specificity            0.9986   0.9994   0.9998   0.9994   1.0000
## Pos Pred Value         0.9964   0.9974   0.9990   0.9969   1.0000
## Neg Pred Value         1.0000   0.9987   0.9994   0.9998   0.9994
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2845   0.1925   0.1738   0.1636   0.1833
## Detection Prevalence   0.2855   0.1930   0.1740   0.1641   0.1833
## Balanced Accuracy      0.9993   0.9971   0.9984   0.9992   0.9986
```

## Plot of Random Forest 


```r
plot(confMatRandForest$table, col=confMatRandForest$byClass, main= paste("Random Forest - Accuracy =" , round(confMatRandForest$overall["Accuracy"], 4)))
```

![](quiz4_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

## Fit Model - GBM Model


```r
set.seed(999)
library(gbm)
```

```
## Warning: package 'gbm' was built under R version 3.6.1
```

```
## Loaded gbm 2.1.5
```

```r
controlGBM<- trainControl(method="repeatedcv", number=5, repeats=1)
modFitGBM<- train(classe~., data=TrainSet, method="gbm", trControl= controlGBM, verbose=FALSE)
modFitGBM$finalModel
```

```
## A gradient boosted model with multinomial loss function.
## 150 iterations were performed.
## There were 53 predictors of which 53 had non-zero influence.
```

## Check the Accuracy of Model - GBM - test data set


```r
predictGBM<- predict(modFitGBM, newdata=TestSet)
confMatGBM<-confusionMatrix(predictGBM, TestSet$classe)
confMatGBM
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1667   13    0    1    0
##          B    7 1111    6    8    5
##          C    0   15 1013   17    2
##          D    0    0    6  938    7
##          E    0    0    1    0 1068
## 
## Overall Statistics
##                                          
##                Accuracy : 0.985          
##                  95% CI : (0.9816, 0.988)
##     No Information Rate : 0.2845         
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.9811         
##                                          
##  Mcnemar's Test P-Value : NA             
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9958   0.9754   0.9873   0.9730   0.9871
## Specificity            0.9967   0.9945   0.9930   0.9974   0.9998
## Pos Pred Value         0.9917   0.9771   0.9675   0.9863   0.9991
## Neg Pred Value         0.9983   0.9941   0.9973   0.9947   0.9971
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2833   0.1888   0.1721   0.1594   0.1815
## Detection Prevalence   0.2856   0.1932   0.1779   0.1616   0.1816
## Balanced Accuracy      0.9962   0.9850   0.9902   0.9852   0.9934
```

##Plot


```r
plot(confMatGBM$table, col=confMatGBM$byClass, main= paste("GBM-Accuracy=", round(confMatGBM$overall["Accuracy"], 4)))
```

![](quiz4_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
##Decision Tree Method


```r
decisionTreeMod<-train(classe~., method="rpart", data= TrainSet)
decisionTreePred<- predict(decisionTreeMod, newdata=TestSet)
confusionMatrix(TestSet$classe, decisionTreePred)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1460   31  182    0    1
##          B  248  410  481    0    0
##          C  183   30  813    0    0
##          D  148  181  589    0   46
##          E   46   85  292    0  659
## 
## Overall Statistics
##                                           
##                Accuracy : 0.5679          
##                  95% CI : (0.5551, 0.5806)
##     No Information Rate : 0.4005          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.4482          
##                                           
##  Mcnemar's Test P-Value : < 2.2e-16       
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.7002  0.55631   0.3449       NA   0.9334
## Specificity            0.9437  0.85839   0.9396   0.8362   0.9183
## Pos Pred Value         0.8722  0.35996   0.7924       NA   0.6091
## Neg Pred Value         0.8516  0.93110   0.6822       NA   0.9902
## Prevalence             0.3543  0.12523   0.4005   0.0000   0.1200
## Detection Rate         0.2481  0.06967   0.1381   0.0000   0.1120
## Detection Prevalence   0.2845  0.19354   0.1743   0.1638   0.1839
## Balanced Accuracy      0.8220  0.70735   0.6423       NA   0.9259
```

##Plot Decisioon Tree


```r
rpart.plot(decisionTreeMod$finalModel)
```

![](quiz4_files/figure-html/unnamed-chunk-13-1.png)<!-- -->


## Final Prediction based on RF Test Data 


```r
predictTEST<- predict(modFitRandForest, newdata= testing)
predictTEST
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```


##Results and Discussions
The overall accuracy of Random Forest Model is better than GBM model. The RandomeForest Model is selected for the final prediction.

## References
Ugulino, W.; Cardador, D.; Vega, K.; Velloso, E.; Milidiu, R.; Fuks, H. Wearable Computing: Accelerometers' Data Classification of Body Postures and Movements. Proceedings of 21st Brazilian Symposium on Artificial Intelligence. Advances in Artificial Intelligence - SBIA 2012. In: Lecture Notes in Computer Science. , pp. 52-61. Curitiba, PR: Springer Berlin / Heidelberg, 2012. ISBN 978-3-642-34458-9. DOI: 10.1007/978-3-642-34459-6_6.
Cited by 2 (Google Scholar)

Read more: http://groupware.les.inf.puc-rio.br/har#ixzz69PPVYt9p
