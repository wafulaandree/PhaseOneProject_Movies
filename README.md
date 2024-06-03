#!/usr/bin/env python
# coding: utf-8

# # Final Project Submission
# 
# Please fill out:
# * Student name: WAFULA SIMIYU
# * Student pace: self paced / part time / full time - PART-TIME
# * Scheduled project review date/time:6/3/2024
# * Instructor name: SAM G/SAM KARU/WINNIE A
# * Blog post URL:https://github.com/wafulaandree/PhaseOneProject_Movies

# #  LIGHTS, CAMERA, DATA

# In[139]:


# Overview:


# In today's digital age, the entertainment industry continues to evolve rapidly, offering new opportunities for tech giants like Microsoft to diversify their portfolio. This project delves into the feasibility of Microsoft venturing into the dynamic world of movies. Leveraging comprehensive datasets from Box Office Mojo, TheMovieDB, and IMDb, we conduct an in-depth analysis to provide actionable insights and recommendations. By examining top-grossing films, prevalent genres, and key players in the industry, we aim to illuminate the path for Microsoft's potential entry into filmmaking. Additionally, we explore the significance of incorporating modern content strategies, such as social media engagement and live streaming, to remain competitive in an ever-changing landscape.

# This is a data driven visionary exploration of Microsoft's potential expansion into the movie industry. As technology continues to redefine entertainment consumption patterns, major players are increasingly diversifying their offerings to stay ahead of the curve. In this project, we embark on a data-driven journey to assess the viability of Microsoft's foray into filmmaking. By analyzing extensive datasets encompassing box office performance, audience preferences, and industry trends, we aim to provide valuable insights that can inform strategic decisions. From uncovering lucrative genres to identifying emerging opportunities in content distribution, our findings aim to equip Microsoft with the knowledge needed to navigate the intricacies of the movie business. Join us as we delve into the intersection of technology and entertainment to envision a future where Microsoft redefines the cinematic landscape.

# # Business Problem

# 
# The key areas of inquiry are as follows:
# 
# What are the highest-grossing movies? These insights will guide Microsoft in identifying successful movie types to replicate.
# How do movies perform across different genres in terms of quantity, revenue, and ratings? This analysis will inform investment decisions by highlighting genres with the most potential.
# What is the significance of a studio's performance in terms of revenue generation? Microsoft may consider collaborating with high-performing studios or directors for content creation.
# Data Understanding:
# Box Office Mojo Dataset:
# This dataset, provided in CSV format, contains information on the domestic and international gross income of movies. It is crucial for evaluating the financial returns of individual movies over time.
# The Movie Database Dataset:
# This dataset offers a snapshot of movie details such as title, genre (identified by ID), language, release date, and voting statistics. It serves as a valuable resource for assessing box office performance and determining the popularity of various genres.
# The IMDb Dataset:
# Comprising multiple CSV files, this dataset includes genre information, reviews, and ratings. Of particular interest are the files imdb.title.basics.csv.gz (which provides genre information and reviews) and imdb.title.ratings.csv.gz.

# In[ ]:





# # Importing necessary modules
# 
# To begin our analysis, we'll import the necessary Python modules. We'll also create aliases for these modules to simplify our code.

# In[140]:


import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
get_ipython().run_line_magic('matplotlib', 'inline')

import seaborn as sns
import sqlite3


# In[141]:


mojo_df = pd.read_csv("zippedData/bom.movie_gross.csv.gz")
mojo_df.head()


# In[142]:


#Displaying part of the dataframe

with pd.option_context('display.max_rows', 20, 'display.max_columns', 5):
    print(mojo_df)
    


# In[143]:


# a summary of the datafarame created and the datatypes, number of columns and rows, null values

mojo_df.info()


# #Description of the above information
# 
# title is the name of the movie.
# 
# studio - the production house.
# 
# foreign_gross and domestic_gross - income in home market and international market.
# 
# year - the year the movie was released.
# 
# note that:
# 1.foreign_gross is a str, should be int.
# 2.missing values are noted in the studio, domestic_gross and foreign_gross columns.

# In[144]:


# Missing Data Management

mojo_df.isnull().sum()


# -The above information allows us to assess the amount of missing data we are working with as follows:
# -For the studio column, due to ths small number the affected rows can be removed as the effect will be insignificant to the overall data.
# -Foreign_gross has the largest number of null values while the studio the smallest number of null values.
# 

# In[145]:


# Studio Column

# checking weight in percentage of the missing data in studio column
mojo_df["studio"].isnull().mean() * 100
0.14762326542663123

# Dropping rows with missing values from the 'studio' column
mojo_df.dropna(subset=['studio'], inplace=True)

# Check if there are any missing values left in the 'studio' column
mojo_df['studio'].isnull().sum()


# In the foreign_gross Column, replace missing values with 0 to show no foreign income.
# 

# In[146]:


mojo_df["foreign_gross"].tail(20)


# In[147]:


mojo_df["foreign_gross"].fillna(0, inplace=True)
mojo_df["foreign_gross"].tail(20)


# For the domestic_gross Column - replace missing values with 0 to show no foreign income.

# In[148]:


mojo_df["domestic_gross"].fillna(0, inplace=True)
mojo_df["domestic_gross"].iloc[926:966]


# In[149]:


mojo_df.isnull().sum()


# In[150]:


#To be able to calculate the income, we will change the datatyoe of foreign_gross column


# In[151]:


mojo_df["foreign_gross"] = pd.to_numeric(mojo_df["foreign_gross"], errors="coerce")
mojo_df.info()


# In[152]:


# displaying the affected rows

fg_missing = mojo_df[mojo_df["foreign_gross"].isna()]
fg_missing


# In[153]:


# creating the new figures to replace NaN
# this will replace the missing figures with correct values

fg_missing_dict = {"Star Wars: The Force Awakens": 1134647993,
                   "Jurassic World": 1018130819, 
                   "Furious 7": 1162334379, 
                   "The Fate of the Furious": 1009996733, 
                   "Avengers: Infinity War": 1373599557}


# In[154]:


# use a for loop to update the values
# it assigns the key to the title and the value to the figure
# locate the key and value in the DataFrame
for index, (key, value) in enumerate(fg_missing_dict.items()):
     mojo_df.loc[mojo_df.title == key, 'foreign_gross'] = value


# In[155]:


# testing the changes
mojo_df[mojo_df['title'] == "The Fate of the Furious"]


# In[156]:


mojo_df.info()
mojo_df.iloc[1868:1873]


# foreign_gross is a str. However it should be an int thus the coversion.

# In[157]:


# converting foreign_gross to int64 for readability

mojo_df["foreign_gross"] = mojo_df["foreign_gross"].astype("int64")
mojo_df.iloc[1868:1873]


# Creating a new column
# The new column is "gross_income" 
# 
# this is done by summing up the "domestic_gross" and "foreign_gross" columns can be beneficial for several reasons:
# 
# Data Consolidation: It consolidates related information into a single column, making it easier to analyze and interpret.
# 
# Analysis Convenience: It simplifies analysis by providing a single metric (gross income) instead of having to consider multiple separate columns.
# 
# Visualization: It enables the visualization of the combined gross income, which may reveal patterns or trends that are not immediately evident when looking at the individual components separately.
# 
# Calculation Efficiency: Pre-calculating the gross income and storing it in a new column can improve computational efficiency, especially if you need to perform calculations or analysis involving gross income frequently.Creating a new column like "gross_income" by summing up the "domestic_gross" and "foreign_gross" columns can be beneficial for several reasons:
# 
# Data Consolidation: It consolidates related information into a single column, making it easier to analyze and interpret.
# 
# Analysis Convenience: It simplifies analysis by providing a single metric (gross income) instead of having to consider multiple separate columns.
# 
# Visualization: It enables the visualization of the combined gross income, which may reveal patterns or trends that are not immediately evident when looking at the individual components separately.
# 
# Calculation Efficiency: Pre-calculating the gross income and storing it in a new column can improve computational efficiency, especially if you need to perform calculations or analysis involving gross income frequently.

# In[158]:


#create a new column 'gross_income'
mojo_df["gross_income"] = mojo_df["domestic_gross"] + mojo_df["foreign_gross"]
mojo_df


# In[159]:


# Convert the "gross_income" column to int64

mojo_df["gross_income"] = pd.to_numeric(mojo_df["gross_income"], errors="coerce", downcast="integer")

mojo_df


# Selecting the columns to keep

# In[160]:


# Reorder the columns
mojo_df = mojo_df.loc[:, ["title", "studio", "gross_income"]]

# Display information about the DataFrame
mojo_df.info()


# Database Dataset
# Read the CSV file and display the first five rows by using the iloc indexer to select the first five rows after reading the file with pd.read_csv():

# In[161]:


# Read the CSV file
moviedb_df = pd.read_csv("zippedData/tmdb.movies.csv.gz", index_col=0)

# Display the first five rows
moviedb_df.iloc[:5]


# In[162]:


# Display summary information from the dataframe created
print(moviedb_df.describe())
print(moviedb_df.info())


# This dataset appears to be well-structured, with no missing values, indicating that it's relatively clean. However, to gain deeper insights, further exploration is needed to understand the content of the genre_ids column. 
# Each integer in this column likely corresponds to a specific genre category, but without additional information, it's challenging to interpret these values accurately. 
# Therefore, a more in-depth analysis is necessary to decode the genre representation within these lists of integers. Additionally, exploring potential relationships between movie genres and other variables could provide valuable insights for subsequent analysis and decision-making

# In[163]:


moviedb_df.isna().sum()


# In[164]:


#select the columns to keep

moviedb_df = moviedb_df[['title', 'vote_average']]
moviedb_df.info()


# In[165]:


# Visualization for mojo_df
plt.figure(figsize=(10, 6))
sns.barplot(x='studio', y='gross_income', data=mojo_df.head(10))
plt.title('Top 10 Studios by Gross Income')
plt.xlabel('Studio')
plt.ylabel('Gross Income')
plt.xticks(rotation=45)
plt.show()


# In[166]:


# Visualization for moviedb_df
plt.figure(figsize=(10, 6))
sns.histplot(moviedb_df['vote_average'], bins=20, kde=True)
plt.title('Distribution of Movie Ratings')
plt.xlabel('Rating')
plt.ylabel('Frequency')
plt.show()


# # The Numbers Dataset
# The Numbers is a film industry data website that tracks box office revenue in a systematic, algorithmic way. This way, the company also conducts research services and forecasts incomes of film project provinding detailed movie financial analysis, including; box office, DVD and Blu-ray sales reports, and release schedules.

# # The IMDb Dataset

# Connecting to dataset

# In[167]:


import zipfile
import sqlite3

# Specify the path to the ZIP file
zip_file_path = "zippedData/im.db.zip"

# Extract the SQLite database file from the ZIP archive
with zipfile.ZipFile(zip_file_path, 'r') as zip_ref:
    zip_ref.extractall("unzipped_data")

# Connect to the SQLite database
try:
    conn = sqlite3.connect("unzipped_data/im.db")
    print("Connected to the database successfully!")
except sqlite3.Error as e:
    print("Error connecting to the database:", e)

# Get a cursor object to execute SQL queries
cur = conn.cursor()

# Execute a SQL query to fetch the list of tables
try:
    cur.execute("SELECT name FROM sqlite_master WHERE type='table';")
    tables = cur.fetchall()
    if tables:
        print("List of tables in the database:")
        for table in tables:
            print(table[0])
    else:
        print("No tables found in the database.")
except sqlite3.Error as e:
    print("Error fetching tables from the database:", e)

# Close the cursor and connection
cur.close()
conn.close()


# In[ ]:





# In[168]:


conn = sqlite3.connect("zippedData/im.db")
cur = conn.cursor()
tables = pd.read_sql("SELECT * FROM sqlite_master WHERE type='table';", conn)
tables


# For this analysis, the movie_basics and movie_ratings tables will be used. They will then be joined using the movie_id column as it is common to both.

# In[169]:


# creating df from movie_basics table
imdb_moviebasics_df = pd.read_sql("SELECT * FROM movie_basics", conn)
imdb_moviebasics_df.head()


# In[170]:


# Function to check if table exists
def table_exists(table_name):
    query = f"SELECT name FROM sqlite_master WHERE type='table' AND name='{table_name}'"
    result = conn.execute(query).fetchone()
    return result is not None

# Check if the table exists before reading from it
if table_exists('movie_basics'):
    imdb_moviebasics_df = pd.read_sql("SELECT * FROM movie_basics", conn)
    print(imdb_moviebasics_df.head())
else:
    print("The table 'movie_basics' does not exist in the database.")


# ## Dataframe: movie_ratings table

# In[171]:


from sqlalchemy import create_engine

# Create SQLAlchemy engine
engine = create_engine('sqlite:///imdb_moviebasics_.db') 

try:
    # Execute SQL query and read data into DataFrame
    query = "SELECT * FROM movie_ratings"
    imdb_movieratings_df = pd.read_sql_query(query, engine)

    # Display the first few rows of the DataFrame
    print(imdb_movieratings_df.head())
except Exception as e:
    print("Error:", e)


# Alternatively:

# In[172]:


imdb_movieratings_df = pd.read_sql("SELECT * FROM movie_ratings", conn)
imdb_movieratings_df.head()


# In[173]:


imdb_movieratings_df.info()


# Data Cleaning for movie_id Column
# Data Cleaning for movie_id Column in imdb_movieratings_df
# Merging DataFrames
# 
# These operations clean and merge the data from the two DataFrames based on the movie_id column, creating a new DataFrame imdb_df containing combined information from both original DataFrames.

# In[175]:


# Check for non-integer values in the 'movie_id' column
non_integer_values = imdb_moviebasics_df[~imdb_moviebasics_df['movie_id'].str.isdigit()]['movie_id']
print(non_integer_values)

# Decide how to handle the non-integer values
# For example, you can drop the rows containing these values
imdb_moviebasics_df = imdb_moviebasics_df[imdb_moviebasics_df['movie_id'].str.isdigit()]

# Convert the 'movie_id' column to type 'int64'
imdb_moviebasics_df['movie_id'] = imdb_moviebasics_df['movie_id'].astype('int64')

# Now, merge the DataFrames as intended
imdb_df = imdb_moviebasics_df.merge(imdb_movieratings_df, on='movie_id', how='inner')


# In[177]:


imdb_df = imdb_df.drop(["original_title", "runtime_minutes", "numvotes"], axis=1)
imdb_df.columns


# In[178]:


imdb_df.info()


# The missing values will be deleted as the movies are not as popular.
# this means that their influence on decision making is not focused on

# Renaming the primary_title column to tile and selecting the columns to keep

# In[179]:


imdb_df.rename(columns = {'primary_title':'title'}, inplace = True)
imdb_df.columns

imdb_df = imdb_df[["title", "start_year", "genres", "averagerating"]]
imdb_df.info()


# # Create One Df

# In[180]:


movies = imdb_df.merge(moviedb_df, on = "title").merge(mojo_df, on = "title")
movies


# In[181]:


movies.columns = movies.columns.str.title()
movies.columns

movies.info()


# In[182]:


#note that the Genres column has missingvalues. 
#This leads us to drop the rows without a genre


# In[183]:


movies = movies.dropna(subset=["Genres"])
movies.info()


# In[184]:


#inclusion of needed columns
movies = movies.assign(Genre = movies["Genres"].str.split(',')).explode("Genre")
movies.head()


# In[185]:


#Combine the Averagerating and Vote_Average columns by getting their average.
#Create a new column Rating for the above
movies["Rating"] = (movies["Averagerating"] + movies["Vote_Average"]) / 2
movies.head()


# In[186]:


# Data Analysis, Visualization and Evaluation

## Top grossing movies


# Calculating the total gross for each film

# In[190]:


top_grossing_movies = movies.sort_values(by = "Gross_Income", ascending=False).head(20)
print(top_grossing_movies[["Title", "Gross_Income"]])


# In[191]:


# Check if the DataFrame is empty
if top_grossing_movies.empty:
    print("The DataFrame 'top_grossing_movies' is empty.")
else:
    # Display the first row of the DataFrame
    print("First row of 'top_grossing_movies' DataFrame:")
    print(top_grossing_movies.head(1))


# top_grossing_movies.iloc[0]

# Avengers: Infinity War grossed the highest income as per the dataset. It is categorised under the Action, Adventure and Sci-Fi genres. The income grossed to USD 2,052,399,557.
# Captain America: Civil War grossed the least amount (USD 1,153,300,000). It is listed in genres similar to Avengers: Infinity War, i.e. Action, Adventure and Sci-F

# In[192]:


# descriptive statistics of the top grossing movies
top_grossing_movies.describe()


# In[193]:


# reset the index

top_grossing_movies.reset_index(inplace=True)

movie_table = top_grossing_movies.style     .background_gradient(cmap='Blues')     .highlight_max(color='grey')     .set_properties(**{'text-align': 'center'})     .set_caption('Top Grossing Movies')     .hide_index()

movie_table


# Grossing by Genre

# In[194]:


genres = movies["Genre"].unique()
print(len(genres))
print(genres)


# In[196]:


genres_gross = movies.groupby("Genre").sum()

genres_gross = genres_gross.sort_values("Gross_Income", ascending=False)

# top 10 genres by total gross income
print(genres_gross[["Gross_Income"]].head(10))


# # # Graph plot

# In[195]:


genres_gross.head(10).plot(kind='bar')
plt.xlabel('Total Gross Income (Billion USD)')
plt.ylabel('Genre')
plt.title('Total Gross Income by Genre')

plt.legend()
plt.show()


# # # Movies with a rating above 5

# In[197]:


rating = movies.groupby("Genre").Rating.agg(["count","mean"]).sort_values(
    'mean', ascending = False)
rating[rating["count"]>=5]


# Datat visualisation

# In[198]:


ax = rating['mean'].head(10).plot(kind='bar', color='blue', width=0.6)

plt.title("Movies by Genre with Rating >5")
plt.xlabel("Genre")
plt.ylabel("Mean Rating")

ax.patch.set_facecolor('grey')

plt.show()


# ## Studio income

# In[ ]:


# gross_per_studio = total gross_income for each studio

gross_per_studio = movies.groupby(["Studio"]).sum()

gross_per_studio = gross_per_studio.sort_values("Gross_Income", ascending=False)

# top 5 studios by total gross income
print(gross_per_studio[["Gross_Income"]].head())

# bottom 5 studios by total gross income
print(gross_per_studio[["Gross_Income"]].tail())


# In[ ]:


gross_per_studio = gross_per_studio.reset_index()
gross_per_studio


# In[ ]:


#Plotting the bar chart to show the top 5 studios
gross_per_studio.head().plot(kind="bar", x="Studio", y="Gross_Income", 
                      figsize=(10,6), legend=None, color="blue")

# set the labels and title of the plot
plt.xlabel("Studio")
plt.ylabel("Gross Income")
plt.title('Top 5 Studios by Gross Earnings')

# show the plot
plt.show()


# ## ASSESMENT
# 
# Throughout this project, I've conducted a comprehensive analysis of various datasets related to the movie industry, aiming to provide valuable insights and recommendations for Microsoft's potential expansion into filmmaking.
# 
# Data Collection and Preparation
# I collected data from multiple sources, including Box Office Mojo, The Movie Database (TMDB), and IMDb, and then performed data cleaning and preprocessing steps to handle missing values, convert data types, and merge relevant datasets.
# 
# Data Analysis
# In my analysis, I identified the highest-grossing movies to understand successful movie types that Microsoft could potentially replicate. Additionally, I delved into genre-wise analysis to evaluate movie performance across different genres in terms of quantity, revenue, and ratings. This analysis helped me highlight genres with the most potential for investment. Moreover, I explored the significance of studio performance in terms of revenue generation, providing insights into potential collaborations or partnerships for content creation.
# 
# Visualization
# To present my findings effectively, I utilized visualizations such as bar plots and histograms. For example, I visualized the top studios by gross earnings and movies by genre with ratings above 5.
# 
# Key Findings
# Based on my analysis, Avengers: Infinity War emerged as the highest-grossing movie, with significant earnings in the Action, Adventure, and Sci-Fi genres. I also identified lucrative genres such as Action, Adventure, and Sci-Fi, indicating potential areas for Microsoft's investment. Additionally, I highlighted top-performing studios, including Buena Vista, Universal Pictures, Fox Studios, Warner Brothers, and Sony Pictures, based on their gross earnings.
# 
# ## Recommendations
# Moving forward, Microsoft could consider replicating successful movie types identified through my analysis, with a focus on genres like Action, Adventure, and Sci-Fi. Exploring collaboration or partnership opportunities with top-performing studios could also be beneficial to leverage their expertise and resources for content creation. By investing in genres with high potential and forging strategic partnerships, Microsoft can enhance its position in the movie industry.
# 
# ## Conclusion
# In conclusion, my analysis provides valuable insights into the movie industry landscape, guiding Microsoft's potential entry into filmmaking. By leveraging data-driven strategies and partnerships, Microsoft can navigate the competitive landscape and capitalize on emerging opportunities in the entertainment industry.


