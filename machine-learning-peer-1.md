What you should submit
----------------------

The goal of your project is to predict the manner in which they did the
exercise. This is the “classe” variable in the training set. You may use
any of the other variables to predict with. You should create a report
describing how you built  
your model, how you used cross validation, what you think the expected
out of sample error is, and why you made the choices you did. You will
also use your prediction model to predict 20 different test cases.

\#\#packages used

    library(ggplot2)
    library(caret)

    ## Warning: package 'caret' was built under R version 4.0.2

    ## Loading required package: lattice

    library(rattle)

    ## Warning: package 'rattle' was built under R version 4.0.2

    ## Loading required package: tibble

    ## Loading required package: bitops

    ## Rattle: A free graphical interface for data science with R.
    ## Version 5.4.0 Copyright (c) 2006-2020 Togaware Pty Ltd.
    ## Type 'rattle()' to shake, rattle, and roll your data.

    library(randomForest)

    ## Warning: package 'randomForest' was built under R version 4.0.2

    ## randomForest 4.6-14

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:rattle':
    ## 
    ##     importance

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

    library(rpart)

collecting the data
-------------------

    training <- read.csv("pml-training.csv",na.strings = c("NA","#DIV/0!", ""))
    testing <- read.csv("pml-testing.csv",na.strings = c("NA","#DIV/0!", ""))

Now clean data sets
-------------------

    # we will remove columns with all na values
    training <- training[,colSums(is.na(training))==0]
    testing <- testing[,colSums(is.na(testing))==0]

    # we will remove unwanted columns like dates and time since they have no role in
    # prediction.
    training <- training[,-c(1:7)]
    testing <- testing[,-c(1:7)]
    ncol(training)

    ## [1] 53

    ncol(testing)

    ## [1] 53

We will create cross validation data sets.
------------------------------------------

    intrain <- createDataPartition(y = training$classe,p=0.7,list = FALSE)
    train_training <- training[intrain,]
    test_training <- training[-intrain,]

now lets see distribution of classes
------------------------------------

    qplot(x = classe,data = train_training,fill= "red")

![](machine-learning-peer-1_files/figure-markdown_strict/unnamed-chunk-5-1.png)

    ## This shows that class are distributed evenly except for the one A class.
    ## Which has relatively more frequency then others.

now lets try distribution tree model.
-------------------------------------

    modfit_rpart <- rpart(classe~.,data = train_training,method = "class")
    pred_rpart <- predict(modfit_rpart,test_training,type = "class")
    confusionMatrix(pred_rpart,as.factor(test_training$classe))

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1535  224   15   91   61
    ##          B   72  798  158   58   64
    ##          C    9   50  735  149   47
    ##          D   45   61   83  640   85
    ##          E   13    6   35   26  825
    ## 
    ## Overall Statistics
    ##                                          
    ##                Accuracy : 0.7703         
    ##                  95% CI : (0.7593, 0.781)
    ##     No Information Rate : 0.2845         
    ##     P-Value [Acc > NIR] : < 2.2e-16      
    ##                                          
    ##                   Kappa : 0.7077         
    ##                                          
    ##  Mcnemar's Test P-Value : < 2.2e-16      
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9170   0.7006   0.7164   0.6639   0.7625
    ## Specificity            0.9071   0.9258   0.9475   0.9443   0.9833
    ## Pos Pred Value         0.7970   0.6939   0.7424   0.7002   0.9116
    ## Neg Pred Value         0.9649   0.9280   0.9406   0.9348   0.9484
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2608   0.1356   0.1249   0.1088   0.1402
    ## Detection Prevalence   0.3273   0.1954   0.1682   0.1553   0.1538
    ## Balanced Accuracy      0.9121   0.8132   0.8319   0.8041   0.8729

now lets try random forest model
--------------------------------

    modfit_rf <- randomForest(as.factor(classe)~.,data = train_training,method = "class")
    pred_rf <- predict(modfit_rf,test_training)
    confusionMatrix(pred_rf,as.factor(test_training$classe))

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1671    6    0    0    0
    ##          B    2 1132    5    0    0
    ##          C    0    1 1017   15    2
    ##          D    0    0    4  949    0
    ##          E    1    0    0    0 1080
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.9939          
    ##                  95% CI : (0.9915, 0.9957)
    ##     No Information Rate : 0.2845          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.9923          
    ##                                           
    ##  Mcnemar's Test P-Value : NA              
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9982   0.9939   0.9912   0.9844   0.9982
    ## Specificity            0.9986   0.9985   0.9963   0.9992   0.9998
    ## Pos Pred Value         0.9964   0.9939   0.9826   0.9958   0.9991
    ## Neg Pred Value         0.9993   0.9985   0.9981   0.9970   0.9996
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2839   0.1924   0.1728   0.1613   0.1835
    ## Detection Prevalence   0.2850   0.1935   0.1759   0.1619   0.1837
    ## Balanced Accuracy      0.9984   0.9962   0.9938   0.9918   0.9990

RESULTS
=======

it is clear from above that random forest model is performing much better then
------------------------------------------------------------------------------

tree model is.
--------------

main prediction
===============

    pred_final <- predict(modfit_rf,testing)
    pred_final

    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
    ##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
    ## Levels: A B C D E
