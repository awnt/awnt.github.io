---
layout: post
title: "Bank Churn Prediction"
subtitle: customer churn forcast using R
background: '/img/posts/bankchurn/euro.jpg'
---
This dataset was provided by [kaggle](https://www.kaggle.com/adammaus/predicting-churn-for-bank-customers){:target="_blank" rel="noopener"}.


### Tools <br />
- R : ggplot,dplyr






#### Data Description and Preparation
```r
# install.packages("tidyverse")
# install.packages("ggpubr")
# install.packages("ggcorrplot")
library(tidyverse)
# tidyverse_update()
library(cowplot)
library(ggpubr)
library(ggcorrplot)
library(dplyr)
```


```r
df<-read.csv("Bank_Churn_Modelling.csv")
str(df)
```

```
## 'data.frame':	10000 obs. of  14 variables:
##  $ RowNumber      : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ CustomerId     : int  15634602 15647311 15619304 15701354 15737888 15574012 15592531 15656148 15792365 15592389 ...
##  $ Surname        : chr  "Hargrave" "Hill" "Onio" "Boni" ...
##  $ CreditScore    : int  619 608 502 699 850 645 822 376 501 684 ...
##  $ Geography      : chr  "France" "Spain" "France" "France" ...
##  $ Gender         : chr  "Female" "Female" "Female" "Female" ...
##  $ Age            : int  42 41 42 39 43 44 50 29 44 27 ...
##  $ Tenure         : int  2 1 8 1 2 8 7 4 4 2 ...
##  $ Balance        : num  0 83808 159661 0 125511 ...
##  $ NumOfProducts  : int  1 1 3 2 1 2 2 4 2 1 ...
##  $ HasCrCard      : int  1 0 1 0 1 1 1 1 0 1 ...
##  $ IsActiveMember : int  1 1 0 0 1 0 1 0 1 1 ...
##  $ EstimatedSalary: num  101349 112543 113932 93827 79084 ...
##  $ Exited         : int  1 0 1 0 0 1 0 1 0 0 ...
```
For basic exploration, the data has no null value.Totally 14 variables was contained as char,int and num.Delete the unnessary variable like Row number ,Customer ID ,Surname).Transform the data to categorical by apply as.factor() with the selected variables as shown below.

```r
df <- df %>%
  dplyr::select(-c(RowNumber,CustomerId,Surname)) %>%
  mutate(Geography=as.factor(Geography),
         Gender=as.factor(Gender),
         Tenure=as.factor(Tenure),
         NumOfProducts=as.factor(NumOfProducts),
         HasCrCard = as.factor(HasCrCard),
         IsActiveMember = as.factor(IsActiveMember),
         Exited=as.factor(Exited))
```




```r
df %>%
  keep(is.numeric)%>%
  gather() %>%
  ggplot(aes(value))+
    facet_wrap(~key,scales="free")+
    geom_histogram(bins=30,color='black')
```

![png](/img/posts/bankchurn/unnamed-chunk-5-1.png)<!-- -->

## Categorical 

**Gender**


```r
Gender_churn<-ggplot(data=df)+
  geom_bar(mapping = aes(Gender,fill=Exited),position='fill',width=0.5)+
  scale_x_discrete("Gender")+
  scale_y_continuous("Percent")+
  guides(fill=guide_legend(title="Churn"))+
  scale_fill_manual(values=c("cornflowerblue","brown"))+
  coord_flip()+
  theme(aspect.ratio = 0.5)
Gender_cat<-ggplot(df,aes(Gender))+
  geom_bar(fill="grey41",width=0.5)+
  coord_flip()+
  theme(aspect.ratio = 0.7)

ggarrange(Gender_churn, Gender_cat,
         ncol = 2, nrow = 1)
```

![png](/img/posts/bankchurn/unnamed-chunk-6-1.png)<!-- -->

**Location**

```r
Location_churn<-ggplot(data=df)+
  geom_bar(mapping = aes(Geography,fill=Exited),position='fill',width=0.5)+
  scale_x_discrete("Geography")+
  scale_y_continuous("Percent")+
  guides(fill=guide_legend(title="Churn"))+
  scale_fill_manual(values=c("cornflowerblue","brown"))
  theme(aspect.ratio = 1)

Location_cat<-ggplot(df,aes(Geography))+
  geom_bar(fill="grey41",width=0.5)+
  theme(aspect.ratio = 0.8)

ggarrange(Location_churn, Location_cat,
         ncol = 2, nrow = 1)
```

![png](/img/posts/bankchurn/unnamed-chunk-7-1.png)<!-- -->


**Number of Products**

```r
Product_churn<-ggplot(data=df)+
  geom_bar(mapping = aes(factor(NumOfProducts),fill=factor(Exited)),position='fill',width=0.8)+
  scale_x_discrete("NumofProducts")+
  scale_y_continuous("Percent")+
  guides(fill=guide_legend(title="Churn"))+
  scale_fill_manual(values=c("cornflowerblue","brown"))+
  theme(aspect.ratio = 0.8)
Product_cat<-ggplot(df,aes(NumOfProducts))+
  geom_bar(fill="grey41",width=0.5)+
  theme(aspect.ratio = 0.8)

ggarrange(Product_churn, Product_cat,
         ncol = 2, nrow = 1)
```

![png](/img/posts/bankchurn/unnamed-chunk-8-1.png)<!-- -->

**Credit Card**

```r
Creditcard_churn<-ggplot(data=df)+
  geom_bar(mapping = aes(factor(HasCrCard),fill=factor(Exited)),position='fill',width=0.5)+
  scale_x_discrete("CreditCard")+
  scale_y_continuous("Percent")+
  guides(fill=guide_legend(title="Churn"))+
  scale_fill_manual(values=c("cornflowerblue","brown"))+
  theme(aspect.ratio = 1)
  
Creditcard_cat<-ggplot(df,aes(factor(HasCrCard)))+
  geom_bar(width=0.5,fill='grey41')+
  theme(legend.position = "left")+
  theme(aspect.ratio = 0.8)

ggarrange(Creditcard_churn, Creditcard_cat,
         ncol = 2, nrow = 1)    
```

![png](/img/posts/bankchurn/unnamed-chunk-9-1.png)<!-- -->
**Active Member**

```r
Active_churn<-ggplot(data=df)+
  geom_bar(mapping = aes(IsActiveMember,fill=Exited),position='fill',width=0.5)+
  scale_x_discrete("IsActiveMember")+
  scale_y_continuous("Percent")+
  guides(fill=guide_legend(title="Churn"))+
  scale_fill_manual(values=c("cornflowerblue","brown"))+
 theme(aspect.ratio = 1)

Active_cat<-ggplot(df,aes(IsActiveMember))+
  geom_bar(width=0.5,fill='grey51')+
  theme(legend.position = "left")+
  theme(aspect.ratio = 0.8)
ggarrange(Active_churn, Active_cat,
         ncol = 2, nrow = 1)
```

![png](/img/posts/bankchurn/unnamed-chunk-10-1.png)<!-- -->

**Tenure**

```r
Tenure_churn<-ggplot(data=df)+
  geom_bar(mapping = aes(Tenure,fill=Exited),position='fill',width=0.5)+
  scale_x_discrete("Tenure")+
  scale_y_continuous("Percent")+
  guides(fill=guide_legend(title="Churn"))+
  scale_fill_manual(values=c("cornflowerblue","brown"))+
 theme(aspect.ratio = 1)

Tenure_cat<-ggplot(df,aes(Tenure))+
  geom_bar(width=0.5,fill='grey51')+
  theme(legend.position = "left")+
  theme(aspect.ratio = 0.8)
ggarrange(Tenure_churn,Tenure_cat,
         ncol = 2, nrow = 1)
```

![png](/img/posts/bankchurn/unnamed-chunk-11-1.png)<!-- -->
## chi-square test of independence

**Gender**

```r
table(df$Gender,df$Exited)
```

```
##         
##             0    1
##   Female 3404 1139
##   Male   4559  898
```

```r
chisq.test(df$Gender,df$Exited,correct=FALSE)
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  df$Gender and df$Exited
## X-squared = 113.45, df = 1, p-value < 2.2e-16
```

**Credit Card**


```r
table(df$HasCrCard,df$Exited)
```

```
##    
##        0    1
##   0 2332  613
##   1 5631 1424
```

```r
chisq.test(df$HasCrCard,df$Exited,correct=FALSE)
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  df$HasCrCard and df$Exited
## X-squared = 0.50948, df = 1, p-value = 0.4754
```
**Num of Products**


```r
table(df$NumOfProducts,df$Exited)
```

```
##    
##        0    1
##   1 3675 1409
##   2 4242  348
##   3   46  220
##   4    0   60
```

```r
chisq.test(df$NumOfProducts,df$Exited,correct=FALSE)
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  df$NumOfProducts and df$Exited
## X-squared = 1503.6, df = 3, p-value < 2.2e-16
```

**Active Member*


```r
table(df$IsActiveMember,df$Exited)
```

```
##    
##        0    1
##   0 3547 1302
##   1 4416  735
```

```r
chisq.test(df$IsActiveMember,df$Exited,correct=FALSE)
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  df$IsActiveMember and df$Exited
## X-squared = 243.76, df = 1, p-value < 2.2e-16
```

**Location*

```r
table(df$Geography,df$Exited)
```

```
##          
##              0    1
##   France  4204  810
##   Germany 1695  814
##   Spain   2064  413
```

```r
chisq.test(df$Geography,df$Exited,correct=FALSE)
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  df$Geography and df$Exited
## X-squared = 301.26, df = 2, p-value < 2.2e-16
```

**Tenure**

```r
table(df$Tenure,df$Exited)
```

```
##     
##        0   1
##   0  318  95
##   1  803 232
##   2  847 201
##   3  796 213
##   4  786 203
##   5  803 209
##   6  771 196
##   7  851 177
##   8  828 197
##   9  771 213
##   10 389 101
```

```r
chisq.test(df$Tenure,df$Exited,correct=FALSE)
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  df$Tenure and df$Exited
## X-squared = 13.9, df = 10, p-value = 0.1776
```
-c(HasCrCard,Tenure)



```r
df_model <- df %>%
  dplyr::select(-c(HasCrCard,Tenure)) 
```



```r
# install.packages('rpart')
# install.packages('partykit')
# install.packages('caTools')
library(caTools)
library(rpart)
library(rpart.plot)
library(party)
library(partykit)
```

Decision Tree

```r
set.seed(123)
sample_data = sample.split(df_model, SplitRatio = 0.75)
train_data <- subset(df_model, sample_data == TRUE)
test_data <- subset(df_model, sample_data == FALSE)

rtree <- rpart(Exited ~ ., data=train_data)
rpart.plot(rtree)
```

![png](/img/posts/bankchurn/unnamed-chunk-20-1.png)<!-- -->



```r
# Predict the values of the test set: pred
pred <- predict(rtree, test_data, type = "class")

# Construct the confusion matrix: conf
confusion <- table(test_data$Exited, pred)

# Print out the accuracy
print(confusion)
```

```
##    pred
##        0    1
##   0 2543  101
##   1  393  296
```

```r
print(sum(diag(confusion)) / sum(confusion))
```

```
## [1] 0.8517852
```
random forest


```r
# install.packages("randomForest")
library(randomForest)
```

```r
rfmod <- randomForest(Exited ~ ., data = train_data, importance = T)
rfmod
```

```
## 
## Call:
##  randomForest(formula = Exited ~ ., data = train_data, importance = T) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 2
## 
##         OOB estimate of  error rate: 13.83%
## Confusion matrix:
##      0   1 class.error
## 0 5163 156  0.02932882
## 1  766 582  0.56824926
```

```r
predTrain <- predict(rfmod, train_data, type = "class")
table(predTrain, train_data$Exited)
```

```
##          
## predTrain    0    1
##         0 5299  453
##         1   20  895
```

```r
predTest <- predict(rfmod, test_data, type = "class")
rfacc <- mean(predTest == test_data$Exited)
rfacc
```

```
## [1] 0.8616862
```

```r
table(predTest, test_data$Exited)
```

```
##         
## predTest    0    1
##        0 2577  394
##        1   67  295
```

```r
# Construct the confusion matrix: conf
confusion <- table(test_data$Exited, predTest)

# Print out the accuracy
print(confusion)
```

```
##    predTest
##        0    1
##   0 2577   67
##   1  394  295
```

```r
print(sum(diag(confusion)) / sum(confusion))
```

```
## [1] 0.8616862
```


```r
full.model <- glm(Exited ~., data = df, family = binomial)
summary(full.model)
```

```
## 
## Call:
## glm(formula = Exited ~ ., family = binomial, data = df)
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -2.5276  -0.5815  -0.3610  -0.1782   3.3052  
## 
## Coefficients:
##                    Estimate Std. Error z value Pr(>|z|)    
## (Intercept)      -2.668e+00  2.791e-01  -9.558   <2e-16 ***
## CreditScore      -6.952e-04  3.036e-04  -2.290   0.0220 *  
## GeographyGermany  9.517e-01  7.299e-02  13.039   <2e-16 ***
## GeographySpain    6.219e-02  7.615e-02   0.817   0.4141    
## GenderMale       -5.238e-01  5.906e-02  -8.870   <2e-16 ***
## Age               7.130e-02  2.766e-03  25.777   <2e-16 ***
## Tenure1          -1.590e-01  1.605e-01  -0.991   0.3218    
## Tenure2          -2.278e-01  1.630e-01  -1.398   0.1622    
## Tenure3          -3.053e-01  1.632e-01  -1.871   0.0614 .  
## Tenure4          -1.094e-01  1.628e-01  -0.672   0.5015    
## Tenure5          -3.042e-01  1.638e-01  -1.857   0.0634 .  
## Tenure6          -1.801e-01  1.634e-01  -1.102   0.2703    
## Tenure7          -3.968e-01  1.653e-01  -2.401   0.0164 *  
## Tenure8          -3.114e-01  1.633e-01  -1.907   0.0565 .  
## Tenure9          -2.483e-01  1.631e-01  -1.522   0.1279    
## Tenure10         -3.367e-01  1.901e-01  -1.771   0.0765 .  
## Balance          -6.680e-07  5.702e-07  -1.172   0.2414    
## NumOfProducts2   -1.544e+00  7.131e-02 -21.651   <2e-16 ***
## NumOfProducts3    2.588e+00  1.802e-01  14.361   <2e-16 ***
## NumOfProducts4    1.636e+01  1.752e+02   0.093   0.9256    
## HasCrCard1       -5.952e-02  6.423e-02  -0.927   0.3540    
## IsActiveMember1  -1.105e+00  6.239e-02 -17.714   <2e-16 ***
## EstimatedSalary   4.410e-07  5.140e-07   0.858   0.3909    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 10109.8  on 9999  degrees of freedom
## Residual deviance:  7421.3  on 9977  degrees of freedom
## AIC: 7467.3
## 
## Number of Fisher Scoring iterations: 14
```

```r
#CreditScore,GenderMale,Age,Balance,NumOfProducts,IsActiveMember
```




