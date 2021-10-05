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
