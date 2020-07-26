# Movies-ETL

### Purpose
This exercise challenged us to create a function that extracts, transforms, and loads data for analysis. This will allow us to perform ETL at anytime, but we also had to anticipate potential issues, which will be outlined later in this report.

### Potential issues

#### Cleaning dropped columns
Any time columns have the potential to be dropped, there may be a function calling for one of those columns. This will throw a KeyError and will stop the script from running. During the creation of this ETL function there were 4 specific instances in which a column could potentially be dropped and the functions designed to clean the data will elicit an error:

1. Box Office
2. Budget
3. Release Date
4. Running Time

While it is unlikely that these columns would be empty, there is a potential that this could occur due to the following code:

``` Python
    wiki_columns_to_keep = [column for column in wiki_movies_df.columns if wiki_movies_df[column].isnull().sum() < len(wiki_movies_df) * 0.9]
    wiki_movies_df = wiki_movies_df[wiki_columns_to_keep]
```

This code will drop any column where 90% or more of the values are null. In the event that any of the afformentioned columns get dropped, there are lines of code that will now draw a KeyError. To bypass this, we used a try-except block on each block of code to bypass this error if the column was dropped.

#### Columns after merging wiki and kaggle metadata
Some of the columns mentioned in the previous section are also used in a function within the function named fill_missing_kaggle_data:

```Python
def fill_missing_kaggle_data(df, kaggle_column, wiki_column):
        df[kaggle_column] = df.apply(
            lambda row: row[wiki_column] if row[kaggle_column] == 0 else row[kaggle_column]
            , axis=1)
        df.drop(columns=wiki_column, inplace=True)
```

If the column in wiki data has been dropped, an error will appear. We used try-except blocks to bypass this potential issue.

#### Keeping columns that may have been dropped
Another major point of concern is the line of code calling for specific columns to keep:

```Python
movies_df = movies_df.loc[:, ['imdb_id','id','title_kaggle','original_title','tagline','belongs_to_collection','url','imdb_link',
                           'runtime','budget_kaggle','revenue','release_date_kaggle','popularity','vote_average','vote_count',
                           'genres','original_language','overview','spoken_languages','Country',
                           'production_companies','production_countries','Distributor',
                           'Producer(s)','Director','Starring','Cinematography','Editor(s)','Writer(s)','Composer(s)','Based on']]
```

Rather than bypassing this potential error, we used the following code, using try-except, to generate a list of values that are available from the original list, in order to continue with the function:

```Python
columns = [col for col in movies_df.columns if col in ['imdb_id','id','title_kaggle','original_title','tagline','belongs_to_collection','url','imdb_link',
                           'runtime','budget_kaggle','revenue','release_date_kaggle','popularity','vote_average','vote_count',
                           'genres','original_language','overview','spoken_languages','Country',
                           'production_companies','production_countries','Distributor',
                           'Producer(s)','Director','Starring','Cinematography','Editor(s)','Writer(s)','Composer(s)','Based on']]
movies_df = movies_df.loc[:,columns]
```

#### Loading to Postgresql
The last step in the ETL process is loading information into our database. In this case, we are using Postgresql. The potential issue here is if there is an existing table, an error will present itself. There is a parameter in the to_sql function that will replace the values in a table (if_exists='replace'). Setting this parameter in such a fashion will prevent the error from appearing.

