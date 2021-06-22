---
layout: post
title: "Bank Churn Prediction"
subtitle: customer churn forecast using R
background: '/img/posts/bankchurn/euro.jpg'
---


This dataset contains customer information,banking products,member status and churn status.To predict whether customers stay or leave,we have to understand their behaviors and identify which patterns are the leaving signs.

[visit kaggle dataset](https://www.kaggle.com/adammaus/predicting-churn-for-bank-customers){:target="_blank" rel="noopener"}

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

From basic exploration, the data has no null values.There are 14 variables contained as char,int and num.Start with discard irrelevant variables including Row number ,Customer ID ,Surname.Then,convert the categorical variables to factor.

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

![png](/img/posts/bankchurn/numeric_hist.png)<!-- -->
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
  select(where(is.numeric),Exited)%>%
  gather(key = "Variable", value = "Value",-Exited) %>%
  ggplot(aes(Value,fill=Exited))+
    facet_wrap(~Variable,scales="free")+
    geom_histogram(bins=30,color='black')+
    scale_fill_manual(values=c("cornflowerblue","brown"))
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


### Testing hypotheses : Chi-Square Test of Independence

Apply hypotheses Chi-square test to determine whether there is a significant association between churn and each factor.

The outcomes represent four variables including Gender,Num of Products,Active members and Locations are indeed useful for churn prediction ,the p-values of which are less than significance level at 0.05.Whereas Tenure and Credit Card have p-values considered significant and that implies the lack of evident related to churning preference.

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
library(caTools)
set.seed(123)
sample_data = sample.split(df_model, SplitRatio = 0.8)
train_data <- subset(df_model, sample_data == TRUE)
test_data <- subset(df_model, sample_data == FALSE)
```
#### Logistic regression
**>> full model**
```r
#Logistic regression(full model)
logis_full<- glm(Exited ~., data = train_data, family = binomial)
summary(logis_full)
anova(logis_full,test='Chisq')
```
```
Call:
glm(formula = Exited ~ ., family = binomial, data = train_data)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.5329  -0.5855  -0.3638  -0.1792   3.2432  

Coefficients:
                   Estimate Std. Error z value Pr(>|z|)    
(Intercept)      -2.986e+00  2.785e-01 -10.721  < 2e-16 ***
CreditScore      -5.757e-04  3.453e-04  -1.667   0.0955 .  
GeographyGermany  9.618e-01  8.236e-02  11.678  < 2e-16 ***
GeographySpain    5.169e-02  8.644e-02   0.598   0.5498    
GenderMale       -5.007e-01  6.689e-02  -7.486 7.09e-14 ***
Age               6.932e-02  3.142e-03  22.061  < 2e-16 ***
Balance          -1.329e-07  6.462e-07  -0.206   0.8371    
NumOfProducts2   -1.501e+00  8.029e-02 -18.696  < 2e-16 ***
NumOfProducts3    2.685e+00  2.099e-01  12.791  < 2e-16 ***
NumOfProducts4    1.635e+01  2.155e+02   0.076   0.9395    
IsActiveMember1  -1.140e+00  7.102e-02 -16.054  < 2e-16 ***
EstimatedSalary   2.482e-07  5.836e-07   0.425   0.6706    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 7829.5  on 7777  degrees of freedom
Residual deviance: 5771.7  on 7766  degrees of freedom
AIC: 5795.7

Number of Fisher Scoring iterations: 14
```

Analysis of Deviance Table
```r
Model: binomial, link: logit

Response: Exited

Terms added sequentially (first to last)


                Df Deviance Resid. Df Resid. Dev  Pr(>Chi)    
NULL                             7777     7829.5              
CreditScore      1     5.65      7776     7823.9   0.01742 *  
Geography        2   245.94      7774     7577.9 < 2.2e-16 ***
Gender           1    81.07      7773     7496.9 < 2.2e-16 ***
Age              1   520.40      7772     6976.5 < 2.2e-16 ***
Balance          1    35.92      7771     6940.5 2.057e-09 ***
NumOfProducts    3   890.43      7768     6050.1 < 2.2e-16 ***
IsActiveMember   1   278.27      7767     5771.8 < 2.2e-16 ***
EstimatedSalary  1     0.18      7766     5771.7   0.67061    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```
Number of Products and Age significantly reduce the residual deviance follow by Geography and IsActivemember ,
while others seem much lower though they have small p-value such as Gender and Balance.


**result**
```r
result_full<-predict(logis_full,test_data,type="response")
predict_full<-ifelse(result_full>0.5,"1","0")
(confusion<-table(predict=predict_full,actual=test_data$Exited))
mean(predict_full==test_data$Exited) #model accuracy
```
```
    actual
predict    0    1
      0 1685  286
      1   72  179
[1] 0.8388839
```

**>> partial model**
using stepwise regression

```r
library(MASS)
logis_step <-logis_full %>% stepAIC(trace = FALSE)
coef(logis_step)
```
```
(Intercept)      CreditScore GeographyGermany   GeographySpain       GenderMale 
   -2.9710715148    -0.0005778074     0.9562957227     0.0525487388    -0.5010858118 
             Age   NumOfProducts2   NumOfProducts3   NumOfProducts4  IsActiveMember1 
    0.0693303513    -1.4961551035     2.6911973442    16.3529508055    -1.1410892969 
```
Apply stepwise to auto select variables.The method drops out 4 of them which relates to chi-square test.

**result**
```r
result_step <- predict(logis_step,test_data,type="response")
predict_step <- ifelse(result_step >0.5,"1","0")
table(Predict=predict_step,Actual=test_data$Exited)
mean(predict_step==test_data$Exited)#accuracy
```

```
      Actual
Predict    0    1
      0 1685  288
      1   72  177
[1] 0.8379838
```
The accuracy of full and partial model both are very closely.It is better for the model to have fewer predictors.
I prefer using the partial one to predict churn rate.

The important variables which are triggers for churn : <br />
No1. Number of Products: risk when the customer use more than two. <br />
No2. Age : Middle age group around 40-50 appears easily to leave. <br />
No3. Active member : It's generally better to have active customers <br />
No4. Location : find some solution to retain Germany customers



##### Future Improvement 
- Plan to develop Decision tree and tune model


###### reference textbook
- Discovering knowledge in data by Daneil T.Larose