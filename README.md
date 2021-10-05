# **Netflix Data Cleaning & EDA**

## Importing Required Libraries


import pandas as pd
import numpy as np

import missingno

import plotly.express as px
import plotly.graph_objects as go

import matplotlib.pyplot as plt

import warnings
warnings.filterwarnings("ignore")

from wordcloud import WordCloud, STOPWORDS

!pip install pycountry
import pycountry 

## Loading the dataset from GitHub

df = pd.read_csv('https://raw.githubusercontent.com/PriyankaMath/Netfix-Data-Cleaning-EDA/main/dataset/netflix_titles.csv')
df1=df.copy()

## **Data Exploration**

df.info()

df.describe(include='all')

## **Data cleaning**

Checking duplicates in show_id column:


show_id_dupl=len(df['show_id'])!=len(set(df['show_id']))
show_id_dupl

Checking for missing values:

df.isna().sum()

Calculating missing data:

for i in df.columns:
    null_rate = df[i].isna().sum()/len(df) * 100
    if null_rate > 0 :
        print("{} null rate: {}%".format(i,round(null_rate,2)))

Plotting Column wise missing values:

missingno.bar(df,fontsize=10,color="dodgerblue",figsize=(10,5))
plt.title('COLUMN WISE MISSING VALUES',fontsize=20)

### **Handling missing values:**
There are missing values in column director, cast, country, date_added and rating.

We can't randomly fill the missing values in columns of director and cast, so we can drop them.

For minimal number of missing values in country and date_added & rating, we can fill them using mode(most common value).

df = df.dropna( how='any',subset=['cast', 'director'])


df['country'] = df['country'].fillna(df['country'].mode()[0])
df['date_added'] = df['date_added'].fillna(df['date_added'].mode()[0])
df['rating'] = df['rating'].fillna(df['country'].mode()[0])

df.isna().sum()

All the missing values in the dataset have either been removed or filled. There are no missing values left.

df.duplicated().sum()

dataset has 0 duplicated values.

### **Renaming the 'listed_in' column as 'Genre' for easy understanding**

df = df.rename(columns={"listed_in":"Genre"})
df['Genre'].head()

### **Adding some new columns:**

Year Added & Month Added

df['date_added'].head()

df['month_added'] = df['date_added'].apply(lambda x: x.split(" ")[0])
df['month_added'].head()

Converting date_added Column to proper datetime format:

df["date_added"] = pd.to_datetime(df['date_added'])
df['date_added'].head()

df['year_added'] = df['date_added'].dt.year
df['year_added'].head()

### **Separating "season" information into a new column from "duration" column:**


df["season"]=""
df["season"] = df[df["duration"].str.contains("Season")]["duration"]

Making "season" column an Integer:

df["season"]=df["season"].fillna("0")
df["season"]=df["season"].str.replace("Season", "").str.replace("s", "") #Removing "season" string from the "season" column
df["season"]=df["season"].astype(str).astype(int)

Making "duration" column an Integer:

df["duration"]=df.duration.str.replace('^(\d+)(.Seasons*)$', "0") #Removing "season" string from the "duration" column
df["duration"]=df["duration"].str.replace(" min", "") ## Remove "min" string from "duration" column
df["duration"]=df["duration"].astype(int) ## Converting duration into Integer


### **Renaming the 'country' column as 'countries' for easy understanding**

Some of the entries under 'country' column have list of countries with comma separated.

df = df.rename(columns={"country":"countries"})
df["countries"]
#df

df

## **Exploratory Data Analysis and Data Visualization**

### **Analysis of Content Type on Netflix:**

content_type_pie = px.pie(df, names='type', height=300, width=600, 
                          title='Movies vs TV Shows',
                          color_discrete_sequence=['#b20710', '#221f1f'])
content_type_pie.update_layout(margin=dict(t=60, b=30, l=0, r=0),
                      paper_bgcolor='#333',
                      title_font=dict(size=25, color='#8a8d93'),
                      font=dict(size=17, color='#8a8d93'))
content_type_pie.show()

### **Content added over the years:**

d1 = df[df["type"] == "TV Show"]
d2 = df[df["type"] == "Movie"]

col = "year_added"

vc1 = d1[col].value_counts().reset_index()
vc1 = vc1.rename(columns = {col : "count", "index" : col})
vc1 = vc1.sort_values(col)

vc2 = d2[col].value_counts().reset_index()
vc2 = vc2.rename(columns = {col : "count", "index" : col})
vc2 = vc2.sort_values(col)

trace1 = go.Scatter(x=vc1[col], y=vc1["count"], name="TV Shows", marker=dict(color="#000000")) 
trace2 = go.Scatter(x=vc2[col], y=vc2["count"], name="Movies", marker=dict(color="#b20710"))

data = [trace1, trace2]

layout = go.Layout(title="Content added over the years",title_font=dict(size=25), legend=dict( orientation="h"))
fig = go.Figure(data, layout=layout)
fig.show()

### **Top Genres**

genres_df=df[['Genre','type']]
genres_df.dropna(subset=['Genre'], inplace=True)
genres_df

Splitting the comma separated "Genre" column values into different columns:

new_genres_df=genres_df['Genre'].str.split(',', expand=True)
new_genres_df

Concatinating the split Genre columns:

splited_genres_df=pd.concat([genres_df, new_genres_df], axis=1)
splited_genres_df.drop(columns=['Genre'], inplace=True)
value=list(splited_genres_df.columns[2:])
splited_genres_df=pd.melt(splited_genres_df, id_vars=['type'], value_vars=value)
splited_genres_df['value']=splited_genres_df['value'].str.strip()
splited_genres_df

Checking for missing values in the new "Genre" data frame created:

print('null value\n', splited_genres_df.isnull().sum(axis=0), '\n')
print('NaN value\n', splited_genres_df.isna().sum(axis=0), '\n')
print(splited_genres_df.shape, '\n')

Handling the missing values in "Genre" data frame:

splited_genres_df.dropna(inplace=True)
splited_genres_df

print('null value\n', splited_genres_df.isnull().sum(axis=0), '\n')
print('NaN value\n', splited_genres_df.isna().sum(axis=0), '\n')
print(splited_genres_df.shape, '\n')

### Word Cloud for Genres

plt.rcParams['figure.figsize'] = (12,12)
text = ' '.join(splited_genres_df['value'])
wordcloud = WordCloud(colormap='RdYlBu', width = 1000,  height = 1000, max_words = 100).generate(text)
plt.imshow(wordcloud)
plt.axis('off')
plt.show()

### **Top Countries**

country_df=df[['countries','type']]
country_df

Splitting the comma separated "countries" column values into different columns:

new_country_df=country_df['countries'].str.split(',', expand=True)
new_country_df

Concatinating the split countries columns:

splited_country_df=pd.concat([country_df, new_country_df], axis=1)
splited_country_df.drop(columns=['countries'], inplace=True)
value=list(splited_country_df.columns[1:])
splited_country_df=pd.melt(splited_country_df, id_vars=['type'], value_vars=value)
splited_country_df['value']=splited_country_df['value'].str.strip()
splited_country_df

Checking for missing values in the new "countries" data frame created:

print('null value\n', splited_country_df.isnull().sum(axis=0), '\n')
print('NaN value\n', splited_country_df.isna().sum(axis=0), '\n')
print(splited_country_df.shape, '\n')

Handling the missing values in "countries" data frame:

splited_country_df.dropna(inplace=True)
print('null value\n', splited_country_df.isnull().sum(axis=0), '\n')
print('NaN value\n', splited_country_df.isna().sum(axis=0), '\n')
print(splited_country_df.shape, '\n')
splited_country_df

country_df=pd.pivot_table(splited_country_df, values='variable', index='value', columns='type', aggfunc=len)
print('null value\n', country_df.reset_index().isnull().sum(axis=0), '\n')
print('NaN value\n', country_df.reset_index().isna().sum(axis=0), '\n')
print(country_df.shape, '\n')
country_df

country_df.replace(to_replace=np.nan, value=0, inplace=True)
print('null value\n', country_df.reset_index().isnull().sum(axis=0), '\n')
print('NaN value\n', country_df.reset_index().isna().sum(axis=0), '\n')
print(country_df.shape, '\n')
country_df

country_df['Total']=country_df['Movie']+country_df['TV Show']
country_df.sort_values(by='Total', ascending=False, inplace=True)
country_df

Generating country code based on country name:

def alpha3code(column):
    CODE=[]
    for country in column:
        try:
            code=pycountry.countries.get(name=country).alpha_3
           # .alpha_3 means 3-letter country code 
            CODE.append(code)
        except:
            CODE.append('None')
    return CODE
# create a column for code 
country_df['CODE']=alpha3code(country_df.index)
country_df.head(20)

print('null value\n', country_df.reset_index().isnull().sum(axis=0), '\n')
print('NaN value\n', country_df.reset_index().isna().sum(axis=0), '\n')
print(country_df.shape, '\n')

top_10 = country_df[:10]
top_10

fig,ax = plt.subplots(figsize=(12,6))
ax.bar(top_10.index, top_10['Total'],color='#b20710')

for s in ['top','left','right']:
    ax.spines[s].set_visible(False)
    
ax.set_title('Top 10 countries on Netflix', fontsize=18, fontweight='bold')

Clearly, the United States, India, and the United Kingdom have a high percentage of content.

We can also express it on the map as follows:

fig = px.choropleth(country_df, locations=country_df.index, 
                    color=country_df['Total'],
                    locationmode='country names',
                    color_continuous_scale=px.colors.sequential.Reds
                   )

fig.update_layout(title='Comparison by country', font=dict(size=18))
fig.show()
