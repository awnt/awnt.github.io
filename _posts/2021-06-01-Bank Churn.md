---
layout: post
title: "Bank Churn Prediction"
subtitle: customer churn forecast using R
background: '/img/posts/bankchurn/euro.jpg'
---


This dataset was provided by [kaggle](https://www.kaggle.com/adammaus/predicting-churn-for-bank-customers){:target="_blank" rel="noopener"}.It contains customer information,banking products,member status and churn status.To predict whether customers stay or leave,we have to understand their behaviors and identify which patterns are the leaving signs.


##### Tools <br />
- R : ggplot,dplyr

 <br />

#### Data Preparation
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

print("null values: ")
colSums(is.na(df)) 
```

```r
'data.frame':	10000 obs. of  14 variables:
$ RowNumber      : int  1 2 3 4 5 6 7 8 9 10 ...
$ CustomerId     : int  15634602 15647311 15619304 15701354  ...
$ Surname        : chr  "Hargrave" "Hill" "Onio" "Boni" ...
$ CreditScore    : int  619 608 502 699 850 645 822...
$ Geography      : chr  "France" "Spain" "France" "France" ...
$ Gender         : chr  "Female" "Female" "Female" "Female" ...
$ Age            : int  42 41 42 39 43 44 50 29 44 27 ...
$ Tenure         : int  2 1 8 1 2 8 7 4 4 2 ...
$ Balance        : num  0 83808 159661 0 125511 ...
$ NumOfProducts  : int  1 1 3 2 1 2 2 4 2 1 ...
$ HasCrCard      : int  1 0 1 0 1 1 1 1 0 1 ...
$ IsActiveMember : int  1 1 0 0 1 0 1 0 1 1 ...
$ EstimatedSalary: num  101349 112543 113932 93827 79084 ...
$ Exited         : int  1 0 1 0 0 1 0 1 0 0 ...

[1] "null values: "
CreditScore       Geography          Gender             Age     
              0               0               0               0 
Tenure            BalanceNum         OfProducts       HasCrCard 
              0               0               0               0     
IsActiveMember EstimatedSalary        Exited               
              0               0               0              
            
```

For basic exploration, the data has no null values.There are 14 variables contained as char,int and num.Start with remove unnecessary variables including Row number ,Customer ID ,Surname.Then,convert the categorical ones to factor as shown.

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

 

### Exploring Numerical variables
The variables appear to be non-normally distributed.Age is right-skewed opposite to CreditScore that is a bit left.While Balance seem to be a normal distribution but has greatest frequency(mode) at 0 . The last, EstimateSalary is symmetical but not normality.

![png](/img/posts/bankchurn/unnamed-chunk-5-1.png)<!-- -->
```r
CreditScore         Age           Balance       EstimatedSalary    
 Min.   :350.0   Min.   :18.00   Min.   :     0   Min.   :    11.58  
 1st Qu.:584.0   1st Qu.:32.00   1st Qu.:     0   1st Qu.: 51002.11  
 Median :652.0   Median :37.00   Median : 97199   Median :100193.91  
 Mean   :650.5   Mean   :38.92   Mean   : 76486   Mean   :100090.24  
 3rd Qu.:718.0   3rd Qu.:44.00   3rd Qu.:127644   3rd Qu.:149388.25  
 Max.   :850.0   Max.   :92.00   Max.   :250898   Max.   :199992.48 
```

Customers at age around 40-50 have higher churn rate than younger. Possibly with their resposibility and maybe more expenses , it's the time to find the best offer preparing for life ,family or elses.It's kind of interesting that customers who have high balance rather leave bank, assume that they might have many accounts on others and just decide to choose the best beneficial one.For Credit score and Salary are not much  difference between churner and non-churner.


![png](/img/posts/bankchurn/boxplot.png)<!-- -->


```
#histogram
df %>%
  keep(is.numeric)%>%
  gather() %>%
  ggplot(aes(value))+
    facet_wrap(~key,scales="free")+
    geom_histogram(bins=30,color='black')
#boxplot
Age_box<-ggplot(data=df)+
  geom_boxplot(mapping = aes(Exited,Age,fill=Exited),width=.5)+
  scale_fill_manual(values=c("cornflowerblue","brown"))
Balance_box<-ggplot(data=df)+
  geom_boxplot(mapping = aes(Exited,Balance,fill=Exited),width=.5)+
  scale_fill_manual(values=c("cornflowerblue","brown"))
CreditScore_box<-ggplot(data=df)+
  geom_boxplot(mapping = aes(Exited,CreditScore,fill=Exited),width=.5)+
  scale_fill_manual(values=c("cornflowerblue","brown"))              
EstimatedSalary_box<-ggplot(data=df)+
  geom_boxplot(mapping = aes(Exited,EstimatedSalary,fill=Exited),width=.5)+
  scale_fill_manual(values=c("cornflowerblue","brown"))
ggarrange(Age_box,Balance_box, CreditScore_box,EstimatedSalary_box,
         ncol = 2, nrow = 2,common.legend = TRUE)
#descriptive
df%>%
  keep(is.numeric)%>%summary()
```

### Exploring Categorical variables

**>> Exited (Churn)** <br />
defined as predicted variable <br /><br />
1 : churn <br />
0 : non-churn 

![png](/img/posts/bankchurn/exited_bar.jpg)<!-- -->
```{r}
ggplot(df,aes(Exited))+
  geom_bar(fill=c("cornflowerblue","brown"),width=0.4)+
  coord_flip()+
  theme(aspect.ratio = 0.3)
```

**>> Gender**<br />
The bank has more male customers than female.Stretch out the bar plot and overlay churn on both categories,the diffences of churn proportion cleary appear that female customers tend to leave more frequently. According to the contingency table in row totals which indicates 25.07% of female churned as compared to male with 16.46%.However,most churners belong to male with 57.25% a bit different to non-churner 42.75%.

![png](/img/posts/bankchurn/gender_bar.jpg)<!-- -->

```
*Two ways table(row totals)*
        Churn
Gender       0     1
  Female 74.93 25.07
  Male   83.54 16.46

```
```
*Two ways table(column totals)*
        Churn
Gender       0     1
  Female 42.75 55.92
  Male   57.25 44.08
```


```r
gender_table<- table(df$Gender,df$Exited,dnn=c('Gender','Churn'))
round(prop.table(gender_table,margin =2),4)*100 #col margin
round(prop.table(gender_table,margin =1),4)*100 #row margin

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



**>> Location**

Without concerning for other circumstances across countries, France and Spain branches have ability to hold customer retention as the small churn rate around 16%.While customers who live in Germany get twice at 32%.


![png](/img/posts/bankchurn/location_bar.jpg)<!-- -->


```
*row totals*                    *column totals*
          Churn                           Churn
Location      0     1           Location      0     1
  France  52.79 39.76             France  83.85 16.15
  Germany 21.29 39.96             Germany 67.56 32.44
  Spain   25.92 20.27             Spain   83.33 16.67

```

```r
location_table<- table(df$Geography,df$Exited,dnn=c('Location','Churn'))
round(prop.table(location_table,margin =2),4)*100 #col margin
round(prop.table(location_table,margin =1),4)*100 #row margin
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




**>> Number of Products**

Customers who involve in 2 products are the highest level of engagement at 92.42%.Although,holding 1 product is not bad but higher risk ,drop to 72.29%.Pay attention to the customers who have 3 or 4 products mostly churn at 82.71% and 100% respectively.Yes,totally 98% non-churners owned only one or two products.This is very curious about the products the banks have provided.

![png](/img/posts/bankchurn/product_bar.jpg)<!-- -->
```
*row totals*                        *column totals*
        Churn                                   Churn
#Product      0      1              #Product     0     1
       1  72.29  27.71                     1 46.15 69.17
       2  92.42   7.58                     2 53.27 17.08
       3  17.29  82.71                     3  0.58 10.80
       4   0.00 100.00                     4  0.00  2.95
```
```r
product_table<- table(df$NumOfProducts,df$Exited,dnn=c('#Product','Churn'))
round(prop.table(product_table,margin =2),4)*100 #col margin
round(prop.table(product_table,margin =1),4)*100 #row margin
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

**Active Member**

Non-active members are more likely leave the bank in comparative to active members.It is reasonable that non-active ones are the major group of churners at 63.92%.

![png](/img/posts/bankchurn/active_bar.jpg)<!-- -->

```
*row totals*                        *column totals*         
      Churn                                 Churn
Active     0     1                   Active     0     1
     0 73.15 26.85                        0 44.54 63.92
     1 85.73 14.27                        1 55.46 36.08

note: 0 for non-active , 1 for active
```

```r
active_table<- table(df$IsActiveMember,df$Exited,dnn=c('Active','Churn'))
round(prop.table(active_table,margin =2),4)*100 #col margin
round(prop.table(active_table,margin =1),4)*100 #row margin
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



**Credit Card**

Base on an equal churn rate,there are no evidences with credit card to indicate the customer behaviors.
However, the bank should find more solutions to retain credit card customers.

![png](/img/posts/bankchurn/creditcard_bar.jpg)<!-- -->
```
*row totals*                           *column totals*           
          Churn                                  Churn 
CreditCard     0     1                  CreditCard     0     1
         0 79.19 20.81                           0 29.29 30.09
         1 79.82 20.18                           1 70.71 69.91
```
```r
creditcard_table<- table(df$HasCrCard,df$Exited,dnn=c('CreditCard','Churn'))
round(prop.table(creditcard_table,margin =2),4)*100 #col margin
round(prop.table(creditcard_table,margin =1),4)*100 #row margin
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

**Tenure**

Tenure groups, like Credit card ,also do not have any clues to predict the customers.

![png](/img/posts/bankchurn/tenure_bar.jpg)<!-- -->

```r
tenure_table<- table(df$Tenure,df$Exited,dnn=c('Tenure','Churn'))
round(prop.table(tenure_table,margin =2),4)*100 #col margin
round(prop.table(tenure_table,margin =1),4)*100 #row margin
chisq.test(df$Tenure,df$Exited,correct=FALSE)
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


### chi-square test of independence
H0: The two variables are independent.
H1: The two variables relate to each other.
We have a chi-squared value of 5.5569. Since we get a p-Value less than the significance level of 0.05, we reject the null hypothesis and conclude that the two variables are in fact dependent.
We have studied about chi-square tests and its parameters with the example in detail. These parameters with examples which we have discussed will help you to correlate with real-life examples based on chi-square-tests.
**Gender**
```
Pearson's Chi-squared test
data:  df$Gender and df$Exited
X-squared = 113.45, df = 1, p-value < 2.2e-16
```

**Credit Card**
```
Pearson's Chi-squared test
data:  df$HasCrCard and df$Exited
X-squared = 0.50948, df = 1, p-value = 0.4754
```
**Num of Products**
```
Pearson's Chi-squared test
data:  df$NumOfProducts and df$Exited
X-squared = 1503.6, df = 3, p-value < 2.2e-16
```
**Active Member**
```
Pearson's Chi-squared test
data:  df$IsActiveMember and df$Exited
X-squared = 243.76, df = 1, p-value < 2.2e-16
```
**Location**
```
Pearson's Chi-squared test
data:  df$Geography and df$Exited
X-squared = 301.26, df = 2, p-value < 2.2e-16
```
**Tenure**
```
Pearson's Chi-squared test
data:  df$Tenure and df$Exited
X-squared = 13.9, df = 10, p-value = 0.1776
```
```r
chisq.test(df$Gender,df$Exited,correct=FALSE)
chisq.test(df$HasCrCard,df$Exited,correct=FALSE)
chisq.test(df$NumOfProducts,df$Exited,correct=FALSE)
chisq.test(df$IsActiveMember,df$Exited,correct=FALSE)
chisq.test(df$Geography,df$Exited,correct=FALSE)
chisq.test(df$Tenure,df$Exited,correct=FALSE)
```

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




