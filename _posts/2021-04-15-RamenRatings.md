---
layout: post
title: "Ramen Ratings"
subtitle: "python"
background: '/img/posts/ramen/unprepared-pasta-bunch-whole-grain-spaghetti-marble-background.jpg'
---

  This project is about to exploring the world of instant ramen.Based on Hans Lienesch's opinion which collected in 'The Big List' dataset. There are 3700 varieties of ramen (up to January 2021)  rated according to brand,taste,packaging style and country. His website [THE RAMEN RATER](https://www.theramenrater.com/){:target="_blank" rel="noopener"} and [youtube channel](https://www.youtube.com/channel/UCoO7I0stFzbrcLbHxZ7tOFA){:target="_blank" rel="noopener"} is so touching for the ramen-craving person.<br />

## Questions <br />
- Which country and brand is the biggest in this market?
- How many kind of packaging and which one is popular?
- What are trending tastes and flavours?


### Tools <br />
- Python : pandas , numpy , matplotlib , seaborn
- Descriptive 
- Word cloud , NMF

## Conclusion <br />
<br />
<br />


#### Data Description and Preparation
The data consists of 5 columns : <br />
1.Review : Reviewer No. <br />
2.Brand : Ramen brand <br />
3.Variety : Describing words including taste , flavour ,noodle style,etc.
Give the big picture about remen <br />
4.Style : Packaging style <br />
5.Stars : as known as Ratings <br />

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Review #</th>
      <th>Brand</th>
      <th>Variety</th>
      <th>Style</th>
      <th>Country</th>
      <th>Stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3702</td>
      <td>Higashimaru</td>
      <td>Seafood Sara Udon</td>
      <td>Pack</td>
      <td>Japan</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3701</td>
      <td>Single Grain</td>
      <td>Chongqing Spicy &amp; Sour Rice Noodles</td>
      <td>Cup</td>
      <td>China</td>
      <td>3.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3700</td>
      <td>Sau Tao</td>
      <td>Seafood Flavour Sichuan Spicy Noodle</td>
      <td>Pack</td>
      <td>Hong Kong</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3699</td>
      <td>Sau Tao</td>
      <td>Jiangnan Style Noodle - Original Flavour</td>
      <td>Pack</td>
      <td>Hong Kong</td>
      <td>4.5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3698</td>
      <td>Sapporo Ichiban</td>
      <td>CupStar Shio Ramen</td>
      <td>Cup</td>
      <td>Japan</td>
      <td>3.5</td>
    </tr>
  </tbody>
</table>
</div>




There is no missing value in 3702 entries but found some differences on 'Stars' column.
Refer to the column apparently is an object type.As I expected it to be a numerical one,I decide to transform the data to float.Yes,after I get rid of the 'Unrated' values that I will turn them to '0'. 

```python
df.info()
df.Stars.unique()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 3702 entries, 0 to 3701
    Data columns (total 6 columns):
     #   Column    Non-Null Count  Dtype 
    ---  ------    --------------  ----- 
     0   Review #  3702 non-null   int64 
     1   Brand     3702 non-null   object
     2   Variety   3702 non-null   object
     3   Style     3702 non-null   object
     4   Country   3702 non-null   object
     5   Stars     3702 non-null   object
    dtypes: int64(1), object(5)
    memory usage: 173.7+ KB

    array([5, 3.5, 4.5, 4, 3.75, 4.25, 3, 3.25, 4.75, 2.5, 2, 0.75, 0, 1.25,
           2.75, 0.5, 1.5, 2.25, 1, 0.25, 1.75, 'Unrated', 1.1, 2.1, 0.9, 3.1,
           4.125, 3.125, 2.125, 2.9, 0.1, 2.8, 3.7, 3.4, 3.6, 2.85, 2.3, 3.2,
           3.65, 1.8], dtype=object)

```python
df.Stars.replace(to_replace={'Unrated':0},inplace=True)
df.Stars=df.Stars.astype(float).round(1)
```
<br />
#### Exploration

##### >> Country

For top 10 countries , Japan is the biggest in the market follow with US and South Korea.Malaysia got 4.2 stars which is the most highest score. The least one belong to Thailand with 3.4 stars.At this point,It concerns that the total amount of noodles is very wide range between maximun and minimum.It could be affected the average stars.


```python
country_s=df.groupby('Country').agg({'Country':'count','Stars':'mean'})
country_s=country_s.rename({'Country':'Total'},axis=1)
country_s.Stars=country_s.Stars.round(1)
country_s = country_s.sort_values(by =['Total','Stars'],ascending = False).reset_index()
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Country</th>
      <th>Total</th>
      <th>Stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Japan</td>
      <td>684</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>1</th>
      <td>United States</td>
      <td>462</td>
      <td>3.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>South Korea</td>
      <td>413</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Taiwan</td>
      <td>372</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>China</td>
      <td>245</td>
      <td>3.5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Thailand</td>
      <td>212</td>
      <td>3.4</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Malaysia</td>
      <td>208</td>
      <td>4.2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Hong Kong</td>
      <td>191</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Indonesia</td>
      <td>161</td>
      <td>4.1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Singapore</td>
      <td>140</td>
      <td>4.1</td>
    </tr>
  </tbody>
</table>
</div>


<br />
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total</th>
      <th>Stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>51.000000</td>
      <td>51.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>72.588235</td>
      <td>3.456863</td>
    </tr>
    <tr>
      <th>std</th>
      <td>140.343319</td>
      <td>0.487136</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.000000</td>
      <td>3.200000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>6.000000</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>53.500000</td>
      <td>3.700000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>684.000000</td>
      <td>4.300000</td>
    </tr>
  </tbody>
</table>
</div>


<br />

```python
sns.barplot(x='Country',y='Total',data=country_s[0:10])
plt.xticks(rotation=45)
plt.show()
```


    
![png](/img/posts/ramen/output_10_0.png)
    



```python
country_s['Stars'].plot(kind='box')
plt.show()
```


    
![png](/img/posts/ramen/output_11_0.png)
    


<br />

```python
annotations=[i for i in country_s[0:9]['Country']]
plt.figure(figsize=(8,6))
plt.scatter('Stars','Total',
           s='Total',
           data=country_s,
           c=country_s.Country.index)
plt.xlabel('Ratings')
plt.ylabel('Total')
for i,label in enumerate(annotations):
    plt.text(country_s.Stars[i],country_s.Total[i],label,color='blue', verticalalignment='bottom',  bbox=dict(facecolor='w',edgecolor = 'w',alpha=0.4))
plt.show()

```


    
![png](/img/posts/ramen/output_12_0.png)
    
```python
annotations=[i for i in country_s[0:10]['Country']]
plt.figure(figsize=(8,6))
plt.scatter('Stars','Total',
           s='Total',
           data=country_s[0:10]) 
plt.xlabel('Ratings')
plt.ylabel('Total')
for i,label in enumerate(annotations):
    plt.text(country_s.Stars[i],country_s.Total[i],label,color='blue', verticalalignment='bottom',  bbox=dict(facecolor='w',edgecolor = 'w',alpha=0.4))
plt.show()
```
![png](/img/posts/ramen/Top10country.png)



```python
sns.histplot(country_s.Stars,bins=10,kde=True)
plt.title('Country vs Stars Distribution')
plt.show()
```
![png](/img/posts/ramen/dist_country.png)



##### >> Brand
'Nissin' has the largest number of flavours far from others with 476 and average rating at 3.9.Marucha,Nongshim,Myojo and Samyang are in a row.The four have nearly amount around 120 along with rating 3.7,4.0,3.9 and 4.1 respectively.We can see that such a slighty different point among the top 5.Anyways,It might meaning a lot if we seriously discover for a really good taste one.


```python
brand_s=df.groupby('Brand').agg({'Brand':'count','Stars':'mean'})
brand_s=brand_s.rename({'Brand':'Total'},axis=1)
brand_s.Stars=brand_s.Stars.round(1)
brand_s = brand_s.sort_values(by =['Total','Stars'],ascending = False).reset_index()
```


```python
brand_s.loc[brand_s.Total>6].sum()
```




    Brand    NissinMaruchanNongshimMyojoSamyang FoodsPaldoM...
    Total                                                 2768
    Stars                                                406.7
    dtype: object




```python
brand_s[1:].describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total</th>
      <th>Stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>543.000000</td>
      <td>543.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.941068</td>
      <td>3.552855</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.054535</td>
      <td>1.101218</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1.000000</td>
      <td>3.200000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.000000</td>
      <td>3.800000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>5.000000</td>
      <td>4.200000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>131.000000</td>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Brand</th>
      <th>Total</th>
      <th>Stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Nissin</td>
      <td>476</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Maruchan</td>
      <td>131</td>
      <td>3.7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nongshim</td>
      <td>119</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Myojo</td>
      <td>111</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Samyang Foods</td>
      <td>103</td>
      <td>4.1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Paldo</td>
      <td>84</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Mama</td>
      <td>71</td>
      <td>3.6</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Sapporo Ichiban</td>
      <td>69</td>
      <td>3.8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Indomie</td>
      <td>56</td>
      <td>4.1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Ottogi</td>
      <td>51</td>
      <td>3.4</td>
    </tr>
  </tbody>
</table>
</div>



```python
annotations=[i for i in brand_s[0:9]['Brand']]
plt.figure(figsize=(8,6))
plt.scatter('Stars','Total',
           s='Total',
           data=brand_s,
           c=brand_s.Brand.index)
plt.xlabel('Ratings')
plt.ylabel('Total')
for i,label in enumerate(annotations):
    plt.text(brand_s.Stars[i],brand_s.Total[i],label,color='blue', verticalalignment='bottom',  bbox=dict(facecolor='w',edgecolor = 'w',alpha=0.4))
plt.show()

```


    
![png](/img/posts/ramen/output_17_0.png)
    

```python
annotations=[i for i in brand_s[0:10]['Brand']]
plt.figure(figsize=(8,6))
plt.scatter('Stars','Total',
           s='Total',
           data=brand_s[0:10])
plt.xlabel('Ratings')
plt.ylabel('Total')
for i,label in enumerate(annotations):
    plt.text(brand_s.Stars[i],brand_s.Total[i],label,color='blue', verticalalignment='bottom',  bbox=dict(facecolor='w',edgecolor = 'w',alpha=0.4))
plt.show()

```

![png](/img/posts/ramen/Top10brand.png)

```python
sns.histplot(brand_s.Stars,bins=10,kde=True)
plt.title('Brand vs Stars Distribution')
plt.show()
```
![png](/img/posts/ramen/dist_brand.png)


#### >> Packaging

```python
style_s = df.groupby('Style').agg({'Style':'count','Stars':'mean'})
style_s=style_s.rename({'Style':'Total'},axis=1).reset_index()
style_s.Stars=style_s.Stars.round(1)
```


```python
style_s.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total</th>
      <th>Stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>8.000000</td>
      <td>8.00000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>462.750000</td>
      <td>3.88750</td>
    </tr>
    <tr>
      <th>std</th>
      <td>723.316321</td>
      <td>0.53569</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>3.50000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.500000</td>
      <td>3.57500</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>110.500000</td>
      <td>3.65000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>674.750000</td>
      <td>3.95000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2095.000000</td>
      <td>5.00000</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.barplot(x='Style',y='Total',data=style_s.sort_values(by='Total',ascending=False))
plt.show()
```


    
![png](/img/posts/ramen/output_20_0.png)
    



```python
sns.displot(df.Stars,bins=10)
plt.show()
```


    
![png](/img/posts/ramen/output_21_0.png)
    

#### Word Cloud
Use the Word cloud to grasp keywords in the variety column and create new categories as Spicy and Flavour.

```python
from os import path
from PIL import Image
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator

word=df['Variety'].apply(lambda x:pd.value_counts(x.split(" "))).sum(axis=0)
```

<h5> >> MEAT Flavour </h5>
 


```python
text = " ".join(desc for desc in df.Variety)
print ("There are {} words in the combination of all review.".format(len(text)))
# Create stopword list:
stopwords = set(STOPWORDS)
stopwords.update(['Flavor','Noodle','Instant','Soup','Artificial','Flavour','Noodles','Ramen','Soba','Udon','Sauce','Tonkotsu','Curry','Bowl','Kimchi','Cup','Yakisoba','Shoyu','Rice'])

# Generate a word cloud image
wordcloud = WordCloud(stopwords=stopwords, background_color="white").generate(text)

# Display the generated image:
# the matplotlib way:
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```
 

    
![png](/img/posts/ramen/new_word_flavour2.png)

Try to cut the spicy out.
    
![png](/img/posts/ramen/delSpicy.png)

Clearly see that Chicken is No.1 and it makes sense.


<h5> >> Soup/Taste </h5>

Additionally , except for meat and spicy theme,there are interesting flavours ,tastes or kinds of soup.Figure out that Curry is the most favorable one,with many types of curry.Tonkotsu is no.2 ,come next with Yakisoba , Miso ,Shoyu and Kimchi which absolutely the the oriental style from Japan and Korea.  

```python
text_F_Other = " ".join(desc for desc in Flavour_Other.Variety)
print ("There are {} words in the combination of all review.".format(len(text)))

# Create stopword list:
stopwords = set(STOPWORDS)
stopwords.update(['flavor','noodle','Instant','big','Soup','vermicelli','Artificial','Taste','Bowl','Style','Soba','Flavour','Rice','Sauce','Udon','Noodles','Ramen','Chili','ramyun','Spicy','Hot','Cup','Bowl'])


# Generate a word cloud image
wordcloud = WordCloud(stopwords=stopwords, background_color="white").generate(text_F_Other)

# Display the generated image:
# the matplotlib way:
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```

    There are 113317 words in the combination of all review.
    


    
![png](/img/posts/ramen/new_word_taste2.png)
    



#### Create new columns
<h5> 1. Spicy </h5>

```python
df['Variety']=df['Variety'].replace('[^a-zA-Z]',' ') #del symbol in column
taste=['Spicy','Chili','Hot','Yum','Curry','Pepper','Mala','Spice','Ginger']
df['Spicy']=df['Variety'].apply(lambda x : 'Spicy' if sum(1 for w in x.split(' ') if w in taste)!=0 else 'NotSpicy')
df.groupby('Spicy').agg({'Stars':'mean','Spicy':'count'})
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Stars</th>
      <th>Spicy</th>
    </tr>
    <tr>
      <th>Spicy</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>NotSpicy</th>
      <td>3.686730</td>
      <td>2826</td>
    </tr>
    <tr>
      <th>Spicy</th>
      <td>3.829795</td>
      <td>876</td>
    </tr>
  </tbody>
</table>
</div>



<h5> 2. Flavours </h5>


```python
FV={'Chicken':'Chicken',
    'Beef':['Beef','Meat','Cow'],
    'Seafood':['Shrimp','Fish','Crab','Seafood','Oyster','Lobster','Goong','Prawn'],
    'Pork':['Pork','Moo','Prok']}

df['Flavour']=df['Variety'].apply(lambda x : list(k for w in x.split(' ') for k,v in FV.items() if w in v))

df=df.explode('Flavour').fillna('Other')
```



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Review #</th>
      <th>Brand</th>
      <th>Variety</th>
      <th>Style</th>
      <th>Country</th>
      <th>Stars</th>
      <th>Spicy</th>
      <th>Flavour</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3702</td>
      <td>Higashimaru</td>
      <td>Seafood Sara Udon</td>
      <td>Pack</td>
      <td>Japan</td>
      <td>5.0</td>
      <td>NotSpicy</td>
      <td>Seafood</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3701</td>
      <td>Single Grain</td>
      <td>Chongqing Spicy &amp; Sour Rice Noodles</td>
      <td>Cup</td>
      <td>China</td>
      <td>3.5</td>
      <td>Spicy</td>
      <td>Other</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3700</td>
      <td>Sau Tao</td>
      <td>Seafood Flavour Sichuan Spicy Noodle</td>
      <td>Pack</td>
      <td>Hong Kong</td>
      <td>5.0</td>
      <td>Spicy</td>
      <td>Seafood</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3699</td>
      <td>Sau Tao</td>
      <td>Jiangnan Style Noodle - Original Flavour</td>
      <td>Pack</td>
      <td>Hong Kong</td>
      <td>4.5</td>
      <td>NotSpicy</td>
      <td>Other</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3698</td>
      <td>Sapporo Ichiban</td>
      <td>CupStar Shio Ramen</td>
      <td>Cup</td>
      <td>Japan</td>
      <td>3.5</td>
      <td>NotSpicy</td>
      <td>Other</td>
    </tr>
  </tbody>
</table>
</div>


Ranking average flavour rating 
```python
Flavour_Other=df.loc[df.Flavour=='Other']
flavour_s=df.groupby('Flavour').agg({'Flavour':'count','Stars':'mean'})
flavour_s=flavour_s.rename({'Flavour':'Total'},axis=1)
flavour_s.Stars=flavour_s.Stars.round(1)
flavour_s = flavour_s.sort_values(by =['Total','Stars'],ascending = False).reset_index()
```



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Flavour</th>
      <th>Total</th>
      <th>Stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Other</td>
      <td>2400</td>
      <td>3.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Chicken</td>
      <td>436</td>
      <td>3.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Seafood</td>
      <td>416</td>
      <td>3.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Beef</td>
      <td>324</td>
      <td>3.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Pork</td>
      <td>151</td>
      <td>3.6</td>
    </tr>
  </tbody>
</table>
</div>


<br/>
<br/>



####  NMF

```python
from sklearn.decomposition import NMF
from sklearn.feature_extraction.text import TfidfVectorizer
temp=df['Variety']
tfidf=TfidfVectorizer(stop_words=['flavor','big','flavour','ramen','noodle','noodles','top','instant','oriental','artificial','taste','bowl','Style','cup','soup','Flavour','Rice',
                                  'Sauce','Noodles','Ramen','ramyun','premium','rice','vermicelli','Cup','udon','with','new','style','yam','green','sauce'])
desc=tfidf.fit_transform(temp)
words = tfidf.get_feature_names()
print(desc.shape)

    (3727, 1886)
```

  


```python
nmf = NMF(n_components=15)
nmf_features = nmf.fit_transform(desc) 

components_temp = pd.DataFrame(nmf.components_,columns=words)

for i in range(0,15):
    components = components_temp.iloc[i,:].nlargest()
    print("group:{}\n{}\n".format(i+1,components))
   

```

    group:1
    chicken     3.713094
    abalone     0.107782
    mushroom    0.091625
    creamy      0.067070
    onion       0.059390
    Name: 0, dtype: float64
    
    group:2
    beef          3.330423
    braised       0.158462
    pho           0.128337
    sauerkraut    0.087606
    stew          0.070613
    Name: 1, dtype: float64
    
    group:3
    spicy      4.364945
    sichuan    0.164546
    king       0.143729
    demae      0.110906
    korean     0.082663
    Name: 2, dtype: float64
    
    group:4
    tom       2.179249
    yum       1.686152
    creamy    0.456325
    kung      0.267612
    goong     0.226173
    Name: 3, dtype: float64
    
    group:5
    curry       2.763459
    white       0.489310
    penang      0.462594
    laksa       0.137008
    malaysia    0.121618
    Name: 4, dtype: float64
    
    group:6
    seafood    2.648500
    xo         0.164790
    chili      0.105372
    demae      0.097479
    neoguri    0.093893
    Name: 5, dtype: float64
    
    group:7
    pork        2.819882
    minced      0.387845
    bone        0.184802
    ribs        0.180931
    mushroom    0.164693
    Name: 6, dtype: float64
    
    group:8
    shrimp    2.581434
    lime      0.095601
    sour      0.092079
    yum       0.070683
    fu        0.062579
    Name: 7, dtype: float64
    
    group:9
    mi            2.156139
    goreng        1.888636
    rasa          0.382159
    ayam          0.343157
    vegetarian    0.334743
    Name: 8, dtype: float64
    
    group:10
    miso        2.708576
    sapporo     0.188684
    rich        0.159935
    japanese    0.146574
    hokkaido    0.129194
    Name: 9, dtype: float64
    
    group:11
    hot          2.603002
    sour         0.735126
    pot          0.139637
    chongqing    0.106733
    buldak       0.094338
    Name: 10, dtype: float64
    
    group:12
    tonkotsu    1.880278
    demae       0.787441
    shoyu       0.595465
    sesame      0.504352
    oil         0.313153
    Name: 11, dtype: float64
    
    group:13
    vegetable     2.363739
    mushroom      0.213656
    pickled       0.165866
    vegetarian    0.151770
    laksa         0.091456
    Name: 12, dtype: float64
    
    group:14
    kimchi        2.358427
    korean        0.341209
    south         0.170860
    mr            0.158396
    vegetarian    0.142083
    Name: 13, dtype: float64
    
    group:15
    yakisoba    2.062222
    japanese    0.316541
    shio        0.249533
    soy         0.197432
    soba        0.187999
    Name: 14, dtype: float64
    
<br />
<br />

## Future Improvement ##
Apply hypothesis tests to figure out which variables significantly affect to average scores,maybe one or more! First,I have planned to use ANOVA but the data appears in nonparametric with its skewness and free distribution.So,I will go for the Kruskal-Wallis.