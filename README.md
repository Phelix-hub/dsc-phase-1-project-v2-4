

# Exploration of Different Types of Films and Their Performance at the Box Office Project


## Project Introduction


This is a data exploration project that in overview, looks at various aspects of Box Office films/movies over time, with the aim 
of getting insights and understanding what would inform successes or failures of movies for a Microsoft's new movie studio. It aims to look at various trends and relationships between different aspects of movies such as existing studios, performance, ratings, budget and review as measures of success or failure



## Data Understanding

In this section of the project, we will work with data from 5 different sources from Bof Office. The Selected Datasets to are csv and tsv flies, which will be read using pandas read_csv methods.

The data used are contained in 5 files available from below 4 websites but now grouped under a single folder called zippedData:

1. [Box Office Mojo](https://www.boxofficemojo.com/) - Website containing records of movie titles with respective studio with gross earnings

2. [Rotten Tomatoes](https://www.rottentomatoes.com/) - Website containing records on Movie info and reviews alongside different ratings alongside respective movie IDs

3. [TheMovieDB](https://www.themoviedb.org/) - Website containing records on movie titles with rating voting information among other records

4. [The Numbers](https://www.the-numbers.com/) - Website containing records on movie titles with respective domestic and gross earnings per movie with Movie ID and release dates


## Business Understanding


In order to help Head of Microsoft's Movie Studio understand movie production market, to aid in knowing what movies to produce and things to consider when running a movie studio, this project aims to answer below questions:

1. What are the most popular Movies in the Box Office?


2. What are the most likely strong Competitors Microsoft is likely to face with in the Movie Studio Business?


3. What are the top 10 most profitable movie studios as per the data files given based on;

    a) most profitable movie
    
    b) average profit for the studios across all movies produced

### Requirements

#### 1. Load the Data with Pandas

Creating individual dataframes from all the datasets selected to be used using pandas library alias as pd

#### 2. Perform Data Cleaning Required to Answer First Question

Performing Data Cleaning in all the files selected individually before merging them into a single dataframe then start answering chosen business Questions

In order to answer the questions , the cleaning would focus on:

* Identify and handle missing values
* Identify and handle text data requiring cleaning

#### 3. Perform Data Aggregation and Cleaning Required to answer questions chosen

Merging at least 3 dataframes into a single dataframe to see the relationship between different categorical and numerical columns drawn from at least 3 datasets to get a clear overview of the Box Office Movie Industry records


Importing different libraries as aliases for purposes of data analysis and exploration


    import pandas as pd
    import numpy as np
    import seaborn as sns
    import matplotlib.pyplot as plt
    import re

    %matplotlib inline

### 1. Box Office Mojo

Below cells load bom.movie_gross.csv.gz as bom_df and preview data in the file for better understanding

    bom_df = pd.read_csv('zippedData/bom.movie_gross.csv.gz')
    bom_df.head()


Getting general information about bom_df dataframe

#to get general info column data types, missing values and general shape of the bom_df dataframe

    bom_df.info() 

Below is the output for above .info()

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 3387 entries, 0 to 3386
    Data columns (total 5 columns):
     #   Column          Non-Null Count  Dtype  
    ---  ------          --------------  -----  
     0   title           3387 non-null   object 
     1   studio          3382 non-null   object 
     2   domestic_gross  3359 non-null   float64
     3   foreign_gross   2037 non-null   object 
     4   year            3387 non-null   int64  
    dtypes: float64(1), int64(1), object(3)
    memory usage: 132.4+ KB
    
#### BOM Data Cleaning

Below subsequent cells cover data cleaning of the bom_df dataset to make it ready for use in the data analysis (EDA)

Looking at the title column, for consistency puporses it makes sense to have the texts uniformly captured with no special characters and same pattern (either capitalized or lower/upper case)

Below cell is a re-usable function that can be called to clean up the title texts


Define regular expression pattern to match special characters

    pattern = r'[^a-zA-Z0-9\s]'

##### Define function to clean text
    def clean_text(text):
        text = re.sub(pattern, '', text)  # remove special characters
        text = re.sub(r'\s+', ' ', text)  # remove extra spaces
        text = text.title()  # convert to lowercase
        text = text.strip()  # remove leading and trailing spaces
        return text

##### Calling above clean_text function to the bom_df title column

    bom_df['title'] = bom_df['title'].apply(clean_text)

From above info, the foreign_gross is an object instead of a floating point. In addition, below cell investigates the number of NaN to determine whether to drop the column.

##### Checking percentage of NaN in the foreign_gross column

    bom_df_percount = (bom_df['foreign_gross'].isna().sum()/len(bom_df['foreign_gross']))*100
    bom_df_percount

Output for the %tage of entries with NaN

        39.85828166519043

With 39.86% of NaN, we drop above column. The studio column are also not significant for subsequent data analysis hence are dropped too

    droppedcol_bom_df = bom_df.drop(['foreign_gross','year','domestic_gross'], axis=1)
    droppedcol_bom_df.info()

Below .info() outputs the remaining columns to confirm that only the title and studio columns remain

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 3387 entries, 0 to 3386
    Data columns (total 2 columns):
     #   Column  Non-Null Count  Dtype 
    ---  ------  --------------  ----- 
     0   title   3387 non-null   object
     1   studio  3382 non-null   object
    dtypes: object(2)
    memory usage: 53.0+ KB


##### dropping duplicates based on the title column

    cleaned_bom_df = droppedcol_bom_df.drop_duplicates(subset='title')

    cleaned_bom_df.value_counts()

        output is as below
            title                 studio 
            10 Cloverfield Lane   Par.       1
            Son Of Sardaar        Eros       1
            Slow West             A24        1
            Smallfoot             WB         1
            Smashed               SPC        1
                                            ..
            Horses Of God         KL         1
            Hostiles              ENTMP      1
            Hot Pursuit           WB (NL)    1
            Hot Tub Time Machine  MGM        1
            Zootopia              BV         1
            Length: 3381, dtype: int64

Applying a lambda function to confirm that entries of movies are unique and no duplicates

    filtered_bom = cleaned_bom_df['title'][cleaned_bom_df.apply(lambda x: x.astype(str).str.contains('Home', case=False)).any(axis=1)]

    print(filtered_bom)


### 2. Rotten Tomatoes 

Below cells load data on Rotten Tomatoes from rt.movie_info.tsv.gz and rt.reviews.tsv.gz for purposes of doing a sneak preview of rotten tomatoes movies info and reviews. We will read both as csv with tab as the delimeter.

Loading rotten tomatoes to a data frame and storing in rt_movie

    rt_movie_info = pd.read_csv('zippedData/rt.movie_info.tsv.gz', sep='\t')
    rt_movie_info.head()

Loading rotten tomatoes to a data frame and storing in rt_reviews

    rt_reviews_info = pd.read_csv('zippedData/rt.reviews.tsv.gz', sep='\t', encoding='unicode_escape')
    rt_reviews_info.head()

Doing a .info() to get general understanding of the datatypes, number of row entries and columns for purposes of data cleaning

    rt_movie_info.info()

        <class 'pandas.core.frame.DataFrame'>
        RangeIndex: 1560 entries, 0 to 1559
        Data columns (total 12 columns):
         #   Column        Non-Null Count  Dtype 
        ---  ------        --------------  ----- 
         0   id            1560 non-null   int64 
         1   synopsis      1498 non-null   object
         2   rating        1557 non-null   object
         3   genre         1552 non-null   object
         4   director      1361 non-null   object
         5   writer        1111 non-null   object
         6   theater_date  1201 non-null   object
         7   dvd_date      1201 non-null   object
         8   currency      340 non-null    object
         9   box_office    340 non-null    object
         10  runtime       1530 non-null   object
         11  studio        494 non-null    object
        dtypes: int64(1), object(11)
        memory usage: 146.4+ KB

#### Rotten Tomattoes Movies Info Data Cleaning

Below cells show various Data cleaning for purposes of subsequent EDA
Below removes/drops duplicates int he rt_movie dataframe using the id Column as the primary Key

    rt_movie_info = rt_movie_info.drop_duplicates(subset='id')

The below code looks at general information about the rt_movie_info df after data removal of duplicates

    rt_movie_info.info()


        <class 'pandas.core.frame.DataFrame'>
        Int64Index: 1560 entries, 0 to 1559
        Data columns (total 12 columns):
         #   Column        Non-Null Count  Dtype 
        ---  ------        --------------  ----- 
         0   id            1560 non-null   int64 
         1   synopsis      1498 non-null   object
         2   rating        1557 non-null   object
         3   genre         1552 non-null   object
         4   director      1361 non-null   object
         5   writer        1111 non-null   object
         6   theater_date  1201 non-null   object
         7   dvd_date      1201 non-null   object
         8   currency      340 non-null    object
         9   box_office    340 non-null    object
         10  runtime       1530 non-null   object
         11  studio        494 non-null    object
        dtypes: int64(1), object(11)
        memory usage: 158.4+ KB
    
looking at number of NaN in the studio column to see if it is useful to keep the column or not which came to 1066

    rt_movie_info['studio'].isna().sum()
With Output as below

      1066

Dropping various columns because of lack of enough data entry and some not being useful for subsequent EDA. This includes the rating column as it does not cover review rating

##### Dropping currency, box_office and studio because the NaN is more than 50%. Also dropping

    rt_movie_droppedcol = rt_movie_info.drop(['currency','studio','box_office','rating','runtime','synopsis'], axis=1)
    rt_movie_droppedcol.info()
output as as below 
   
       <class 'pandas.core.frame.DataFrame'>
        Int64Index: 1560 entries, 0 to 1559
        Data columns (total 6 columns):
         #   Column        Non-Null Count  Dtype 
        ---  ------        --------------  ----- 
         0   id            1560 non-null   int64 
         1   genre         1552 non-null   object
         2   director      1361 non-null   object
         3   writer        1111 non-null   object
         4   theater_date  1201 non-null   object
         5   dvd_date      1201 non-null   object
        dtypes: int64(1), object(5)
        memory usage: 85.3+ KB

Given that 3 of the datasets have date columns, below function is created to be used to convert the date columns into pandas datetime format so that dates are properly intepreted 

    def clean_date(df, date_col):
        """Converts a date column from object to datetime format"""

        # Convert column to datetime format
        df[date_col] = pd.to_datetime(df[date_col], errors='coerce')

        # Return cleaned DataFrame
        return df

Calling the clean_date function to change theater_date and dvd_date columns to datetime format

    rt_movie_droppedcol = clean_date(rt_movie_droppedcol,'theater_date')
    rt_movie_droppedcol = clean_date(rt_movie_droppedcol,'dvd_date')

Viewing general information on the rt_movie_droppedcol dataframe after cealning the dvd_theater date columns using clean_date function

    rt_movie_droppedcol.info()

        <class 'pandas.core.frame.DataFrame'>
        Int64Index: 1560 entries, 0 to 1559
        Data columns (total 6 columns):
         #   Column        Non-Null Count  Dtype         
        ---  ------        --------------  -----         
         0   id            1560 non-null   int64         
         1   genre         1552 non-null   object        
         2   director      1361 non-null   object        
         3   writer        1111 non-null   object        
         4   theater_date  1201 non-null   datetime64[ns]
         5   dvd_date      1201 non-null   datetime64[ns]
        dtypes: datetime64[ns](2), int64(1), object(3)
        memory usage: 85.3+ KB

#### Rotten Tomatoes Reviews Data Cleaning

This cleans the 3rd dataset which is also on Rotten Tomatoes reviews 

Starting with doing a .head() function to see 5 top rows of the previously loaded dataset

    rt_reviews_info.head()

To preview the fist row of the review column

    rt_reviews_info.iloc[0]['review']

with below output
    "A distinctly gallows take on contemporary financial mores, as one absurdly rich man's limo ride across town for a haircut functions as a state-of-the-nation discourse. "

doing .info() to get general info about rt_reviews_info

    rt_reviews_info.info() 

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 54432 entries, 0 to 54431
    Data columns (total 8 columns):
     #   Column      Non-Null Count  Dtype 
    ---  ------      --------------  ----- 
     0   id          54432 non-null  int64 
     1   review      48869 non-null  object
     2   rating      40915 non-null  object
     3   fresh       54432 non-null  object
     4   critic      51710 non-null  object
     5   top_critic  54432 non-null  int64 
     6   publisher   54123 non-null  object
     7   date        54432 non-null  object
    dtypes: int64(2), object(6)
    memory usage: 3.3+ MB
    
Cleaning the date column to datetime format

    rt_reviews_info = clean_date(rt_reviews_info,'date')
    rt_reviews_info

Dropping a few unwanted columns

    rt_reviews_info = rt_reviews_info.drop(['review','rating','critic','publisher','date'], axis=1)

Doing a .describe to get a statistical summary of the data on remaining columns

    rt_reviews_info['fresh'].describe()

Output

    count     54432
    unique        2
    top       fresh
    freq      33035
    Name: fresh, dtype: object

Removing duplicates in the rt_reviews_info dataframe

    rt_reviews_info = rt_reviews_info.drop_duplicates(subset='id')


### 3. TheMovieDB

In below cells, we explore TheMovieDB in details to understand more about it for purposes of EDA. We start by reading the csv file into a pandas Dataframe called tmdb_movies_df.

We then get more info on this dataset with in order to identify the primary key to use for purposes of later merging it with other dataset dataframes 

##### Loading The MovieDB datafile to a data frame and storing in tmdb_movies_df and doing a view of top rows

    tmdb_movies_df = pd.read_csv('zippedData/tmdb.movies.csv.gz')
    tmdb_movies_df.head()

Doing .info() to get general info about tmdb_movies_df 

    tmdb_movies_df.info() 

With below output

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 26517 entries, 0 to 26516
    Data columns (total 10 columns):
     #   Column             Non-Null Count  Dtype  
    ---  ------             --------------  -----  
     0   Unnamed: 0         26517 non-null  int64  
     1   genre_ids          26517 non-null  object 
     2   id                 26517 non-null  int64  
     3   original_language  26517 non-null  object 
     4   original_title     26517 non-null  object 
     5   popularity         26517 non-null  float64
     6   release_date       26517 non-null  object 
     7   title              26517 non-null  object 
     8   vote_average       26517 non-null  float64
     9   vote_count         26517 non-null  int64  
    dtypes: float64(2), int64(3), object(5)
    memory usage: 2.0+ MB
    
#### TheMovieDB Data Cleaning

Below cells cover cleaning of this tmdb_movies_df by dropping unnecessary columns and rows, handling missing values, formating texts and row entries into the right data types

##### Dropping a few columns deemed unnecessary for data exploration phase

    tmdb_coldropped_df = tmdb_movies_df.drop(['genre_ids','id','Unnamed: 0','original_language','original_title','vote_count'], axis=1)
    tmdb_coldropped_df

Calling the clean_text function previously defined to clean and format texts in the title column

    tmdb_coldropped_df['title'] = tmdb_coldropped_df['title'].apply(clean_text)

Calling the clean_date function prevoiusly defined to clean and format the release_date column

    tmdb_coldropped_df = clean_date(tmdb_coldropped_df, 'release_date')

Doing .info to get general info and confirm change of datatype for the release_date column

    tmdb_coldropped_df.info()
below is the output

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 26517 entries, 0 to 26516
    Data columns (total 4 columns):
     #   Column        Non-Null Count  Dtype         
    ---  ------        --------------  -----         
     0   popularity    26517 non-null  float64       
     1   release_date  26517 non-null  datetime64[ns]
     2   title         26517 non-null  object        
     3   vote_average  26517 non-null  float64       
    dtypes: datetime64[ns](1), float64(2), object(1)
    memory usage: 828.8+ KB

Checking if the multiple entries of specific movies are duplicates and not unique entries

    filtered_tmdb = tmdb_coldropped_df['title'][tmdb_coldropped_df.apply(lambda x: x.astype(str).str.contains('Home', case=False)).any(axis=1)]


Dropping duplicate rows based on title column and doing value count to see frequency of unique identifies

    cleaned_tmdb_df = tmdb_coldropped_df.drop_duplicates(subset='title')
    cleaned_tmdb_df['title'].value_counts()

        Harry Potter And The Deathly Hallows Part 1    1
        Love Taxes                                     1
        Followers                                      1
        The Taking Of Ezra Bodine                      1
        Andrew Dice Clay Presents The Blue Show        1
                                                      ..
        Wish You Were Here                             1
        We Steal Secrets The Story Of Wikileaks        1
        Bastards                                       1
        Petes Christmas                                1
        The Church                                     1
        Name: title, Length: 24631, dtype: int64

### The Movie DB

Next is to read data from TheMovieDB CSV file into a pandas dataframe and conduct various data preview and data cleaning before EDA

Loading The Number datafile to a data frame and storing in tn_movie_budgets_df and doing a view a few top rows

    tn_movie_budgets_df = pd.read_csv('zippedData/tn.movie_budgets.csv.gz')
    tn_movie_budgets_df.head()

Doing .info() to get general info about tn_movie_budgets_df

    tn_movie_budgets_df.info()
    
output as below

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5782 entries, 0 to 5781
    Data columns (total 6 columns):
     #   Column             Non-Null Count  Dtype 
    ---  ------             --------------  ----- 
     0   id                 5782 non-null   int64 
     1   release_date       5782 non-null   object
     2   movie              5782 non-null   object
     3   production_budget  5782 non-null   object
     4   domestic_gross     5782 non-null   object
     5   worldwide_gross    5782 non-null   object
    dtypes: int64(1), object(5)
    memory usage: 271.2+ KB
    


#### The Numbers Data Cleaning

Below cells cover cleaning of this tn_movies_budgets_df by dropping unnecessary columns and rows, handling missing values, formating texts and row entries into the right data types

Making the id column be the index column

    tn_movie_budgets_df = tn_movie_budgets_df.set_index('id')

calling the clean_text function previously defined to clean and format texts in the production_budget, domestic_budget and worldwide_gross columns and then convert them to floating points

    tn_movie_budgets_df['production_budget'] = tn_movie_budgets_df['production_budget'].apply(clean_text).astype(float)
    tn_movie_budgets_df['domestic_gross'] = tn_movie_budgets_df['domestic_gross'].apply(clean_text).astype(float)
    tn_movie_budgets_df['worldwide_gross'] = tn_movie_budgets_df['worldwide_gross'].apply(clean_text).astype(float)
    tn_movie_budgets_df['movie'] = tn_movie_budgets_df['movie'].apply(clean_text)

Dropping the release date column since it this df will be merged later with another df with release_date 
    tn_movie_budgets_df = tn_movie_budgets_df.drop(['release_date'], axis=1)

    tn_movie_budgets_df

Checking general information about the tn_movie_budget_df to check data type, and existence of non-null counts in the columns

    tn_movie_budgets_df.info()
Output as below 

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 5782 entries, 1 to 82
    Data columns (total 4 columns):
     #   Column             Non-Null Count  Dtype  
    ---  ------             --------------  -----  
     0   movie              5782 non-null   object 
     1   production_budget  5782 non-null   float64
     2   domestic_gross     5782 non-null   float64
     3   worldwide_gross    5782 non-null   float64
    dtypes: float64(3), object(1)
    memory usage: 225.9+ KB
    
Renaming the movie column to title to match other dataframes for purposes of better future merging

    tn_movie_budgets_df = tn_movie_budgets_df.rename(columns={'movie': 'title'})

    tn_movie_budgets_df

From above value_counts() Method it is evident that there are duplicate entries on same movies. Below cell confirms if those duplicates are genuine duplicates and not different movies

Checking if the multiple entries of specific movies are duplicates and not unique entries

    filtered_tn = tn_movie_budgets_df['title'][tn_movie_budgets_df.apply(lambda x: x.astype(str).str.contains('Home', case=False)).any(axis=1)]

    print(filtered_tn)
    ```

        id
        99                          Spiderman Homecoming
        44                                          Home
        30    Miss Peregrines Home For Peculiar Children
        37                             Home On The Range
        5                                  Daddys Home 2
        41                               A Dogs Way Home
        88                                   Daddys Home
        55                            Sweet Home Alabama
        85                   Welcome Home Roscoe Jenkins
        32                  Star Trek Iv The Voyage Home
        29                                     Homefront
        89                 Home Alone 2 Lost In New York
        6                          Home For The Holidays
        48                          Take Me Home Tonight
        70                                  The Homesman
        92                                    Home Alone
        57                                    Home Again
        97                                    Home Fries
        56                                          Home
        4                       A Prairie Home Companion
        65                        Jeff Who Lives At Home
        2                 A Home At The End Of The World
        72                                   Coming Home
        32                    Son Of Rambow A Home Movie
        66                                      Home Run
        60                                          Home
        16                                   Home Movies
        Name: title, dtype: object


Removing Duplicate entries based on movie column

    cleaned_tnmovie_budgets_df = tn_movie_budgets_df.drop_duplicates(subset='title')
    cleaned_tnmovie_budgets_df['title'].value_counts()


        Avatar                1
        Ultramarines          1
        Glitter               1
        Bright Star           1
        Club Dread            1
                             ..
        The Age Of Adaline    1
        Glory Road            1
        John Wick             1
        Pokemon 2000          1
        My Date With Drew     1
        Name: title, Length: 5698, dtype: int64

Doing a .describe to get statistical measures of the columns with numerical values

    cleaned_tnmovie_budgets_df.describe()

#### Merging of Different Data Frames

Below cells merging of above cleaned the dataframes in order to bring up needed columns for EDA

Merging the Rotten Tomatoes Movie Info and Reviews DataFrames

Merging two dfs from rotten tomatoes files

    rt_merged_df = pd.merge(rt_reviews_info, rt_movie_droppedcol, on='id')
    rt_merged_df

Doing .info method on the rotten tomatoes data merge

    rt_merged_df.info()

        <class 'pandas.core.frame.DataFrame'>
        Int64Index: 1135 entries, 0 to 1134
        Data columns (total 8 columns):
         #   Column        Non-Null Count  Dtype         
        ---  ------        --------------  -----         
         0   id            1135 non-null   int64         
         1   fresh         1135 non-null   object        
         2   top_critic    1135 non-null   int64         
         3   genre         1133 non-null   object        
         4   director      1014 non-null   object        
         5   writer        891 non-null    object        
         6   theater_date  996 non-null    datetime64[ns]
         7   dvd_date      996 non-null    datetime64[ns]
        dtypes: datetime64[ns](2), int64(2), object(4)
        memory usage: 79.8+ KB
    

## Exploratory Data Analysis (EDA)


After looking at all the above cleaned Data from different datasets, below EDA will focus on 3 datasets;
1. The Box Office Mojo
2. The Movie DB
3. The Numbers

The two datasets on Rotten Tomatoes (rotten tomatoes movie info and rotten tomatoes reviews), were ignoted because of lack of specific primary key column that could be used to relate them with the above chosen 3. Otherwise, above 3 have 'title' column as the primary key column to use to call on the .merge method


Doing a merge between Box Office Mojo and The Numbers using 'title' column to consolidate the gross and budget datasets

    bomtn_budget_mergeddf = pd.merge(cleaned_bom_df, cleaned_tnmovie_budgets_df, on='title')
    bomtn_budget_mergeddf


Because above bomtn_budget_mergeddf, which is a merge between Box Office Mojo and The Numbers datasets to summarize budgets have a title column, and cleaned_tmdb_df from The MoviesDB, which has the ratings also have title column, It makes sense to merge this with the previously merged bomtn_budget_mergeddf to generate a dataframe with now 3 merged datasets


Merging the 3 datasets from Box Office Mojo,The Numbers and The Movie DB
to add the 'studio','popularity' and 'vote_average' columns

    bom_tn_tmdb_mergeddf  = pd.merge(bomtn_budget_mergeddf, cleaned_tmdb_df, on='title')
    bom_tn_tmdb_mergeddf


Getting general info about the merged dataset

    bom_tn_tmdb_mergeddf.info()

        <class 'pandas.core.frame.DataFrame'>
        Int64Index: 1271 entries, 0 to 1270
        Data columns (total 8 columns):
         #   Column             Non-Null Count  Dtype         
        ---  ------             --------------  -----         
         0   title              1271 non-null   object        
         1   studio             1270 non-null   object        
         2   production_budget  1271 non-null   float64       
         3   domestic_gross     1271 non-null   float64       
         4   worldwide_gross    1271 non-null   float64       
         5   popularity         1271 non-null   float64       
         6   release_date       1271 non-null   datetime64[ns]
         7   vote_average       1271 non-null   float64       
        dtypes: datetime64[ns](1), float64(5), object(2)
        memory usage: 89.4+ KB
    

In order to get an understanding of which movies make profits and by how much, we need to add a revenue column, which is the difference between worldwide gross and the production_budget. This assumes that worldwide_gross is the gross income from the sale of the movies (a summation of domestic and foreign gross income)


Adding a 'revenue column' in the merged df as 'worldwide_gross' minus 'production_budget'. Figues in USD

    bom_tn_tmdb_mergeddf["revenue"] = bom_tn_tmdb_mergeddf["worldwide_gross"] - bom_tn_tmdb_mergeddf["production_budget"]

    bom_tn_tmdb_mergeddf.head()

### Answering Bussiness Understanding Questions Based on EDA


#### 1. The Most Popular Movies in the Box Office

Below cells tend to inquire on the most popular movies based on the merged dataset to answer the 1st Question in the Business Understanding


Group by the 'title' column and get most popular movies

    most_popular_movie = bom_tn_tmdb_mergeddf.groupby('title').mean()

Sort the resulting DataFrame in descending order of the maximum values

    most_popular_movie_sorted = most_popular_movie.sort_values('popularity', ascending=False)
    most_popular_movie_sorted.head(10)

The most Popular Movies in the Box Office based on above code are:
1. Avengers Infinity War
2. John Wick
3. The Hobbit The Battle Of The Five Armies
4. Guardians Of The Galaxy
5. Blade Runner 2049
6. Fantastic Beasts The Crimes Of Grindelwald
7. Ralph Breaks The Internet
8. Spiderman Homecoming
9. Antman And The Wasp
10. Avengers Age Of Ultron

#### 2. Microsoft's likely strong competitors

What are the most likely strong Competitors Microsoft is likely to face with in the Movie Studio Business?

Below cells tend to answer business understanding question 2 above


Group by the 'studio' column and get the average ratings of all movies per studio

    most_popular_studio = bom_tn_tmdb_mergeddf.groupby('studio').mean()

Sort the resulting DataFrame in descending order of the maximum values

    most_popular_studio_sorted = most_popular_studio.sort_values('popularity', ascending=False)
    most_popular_studio_sorted

Ploting a bar graph of most popular Movie studios based on movie popularity average per studio

    most_popular_studio_sorted['popularity'].head(10).plot(kind='bar')

Add a title and axis labels

    plt.title('Movie Studios Popularity')
    plt.xlabel('Top 10 Movie Studios')
    plt.ylabel('popularity')

Display the graph

plt.show()
    
    This would show a graph with above plot

Based on above analysis, Microsoft's most likely top ten strong competitors are as below. This is based on popularity of top movies produced
1. BV
2. MGM
3. LGP
4. Amazon
5. LG/S
6. WB
7. Fox
8. STX
9. Sony
10. Amapurna

#### 3. The top 10 most profitable movie studios

Below Cells tend to answer the 3rd question on 

What are the top 10 most profitable movie studios as per the data files given based on;

a) most profitable movie

b) average profit for the studios across all movies produced



##### 3(a) 10 most profitable movie studios based on most Profititable Movie 


Group by the 'studio' column and get the maximum value for each group

    max_values = bom_tn_tmdb_mergeddf.groupby('studio').max()

Sort the resulting DataFrame in descending order of the maximum values

    max_values_sorted = max_values.sort_values('revenue', ascending=False)
    max_values_sorted.head(10)

Ploting a bar graph of most profitable production studio (top 10) based on maximum revenue from most profitable movie

    max_values_sorted['revenue'].head(10).plot(kind='bar')

Add a title and axis labels

    plt.title('Movie Studios vs Revenue by Most Profitable Movie')
    plt.xlabel('Top 10 Movie Studios')
    plt.ylabel('Revenue in USD')

Display the graph

plt.show()
   
     This would show a graph for the above plot


To Answer 3(a) question on top ten most profitable studios based on the most profitable movie. It is evident that BV (Buena Vista which is a brand name that has historically been used for divisions and subsidiaries of The Walt Disney Company) is the most profitable studio based on the most profitable movie 'Zootopia'. Below is the full top 10 list
1. BV
2. Uni
3. WB
4. P/DW
5. Sony
6. Par
7. Fox
8. WB (NL)
9. LGF
10. LG/S


##### 3(b) 10 most profitable movie studios based on Average Profit for all Movies Produced


Group by the 'studio' column and get the mean value for each group

    mean_values = bom_tn_tmdb_mergeddf.groupby('studio').mean()

Sort the resulting DataFrame in descending order of the mean values

    mean_values_sorted = mean_values.sort_values('revenue', ascending=False)
    mean_values_sorted

Ploting a bar graph of most profitable production studios (top 10) based on average revenue from all movies produced in that studio

    mean_values_sorted['revenue'].head(10).plot(kind='bar')

Add a title and axis labels

    plt.title('Movie Studios vs Average Revenue per studio')
    plt.xlabel('Top 10 Movie Studios')
    plt.ylabel('Revenue in USD')

Display the graph

    plt.show()

    This would show a graph with above plot
    

To Answer 3(b) question on top ten most profitable studios based on average revenues. It is evident below are the most profitable movie studio based on average revenues generated on movies produced from top;
1. P/DW
2. BV
3. GrtIndia
4. Uni
5. Fox
6. Sony
7. WB(NL)
8. WB
9. UTV
10. Strand

## Conclusion & Recommendations

Based on above analysis, below are the recommendations for Head of Microsoft Studio:

1. Considering the Top 10 most popular movies based on maximum revenue generation, it is evident that most popular movies do not necessarily translate to top revenue generating. However, popularity scores are critical in knowing market penetration (studio's existence awareness)

2. With this analysis, Head of Microsoft Movie Studio now knows the top 10 most profitable movie studios, which can be considered as major competitors. It is therefore, recommended that Microsoft focus on above mentioned 10 for purposes of competitor analysis (business models,marketing models, and SWOT analysis)

3. In future analysis of Studios performance, focus should be on the average revenue based on movies' financial performances. This is evidently highlighted in the two analyses where ratings based on most revenue generating movies gives a different top 10 studios compared to that from the overall average revenues generated from all the movies from a single studio

4. As far as future analyses are concerned, data collected from Rotten Tomatoes, which better cover ratings and reviews, should include a the movie titles column so as to be efectively have definte way of relating them with other datasets with movie titles. This is one of the surest ways of merging ratings to their respective movies in other datasets. The Rotten Tomatoes, despite having better reviews data, had to be dropped because they both lacked the primary key column that matches the other 3 data sets
