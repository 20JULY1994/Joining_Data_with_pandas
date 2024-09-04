# Joining_Data_with_pandas

Run the hidden code cell below to import a few of the datasets used in this course.

> [!Note] 
> There are a large number of datasets in the `datasets/` folder. Many of these are Pickle files, which you can read using `pd.read_pickle(path_to_file)`. An example is included in the cell below.

 ```python
# Import pandas
import pandas as pd

# Import some of the course datasets 
actors_movies = pd.read_csv("datasets/actors_movies.csv")
business_owners = pd.read_pickle("datasets/business_owners.p")
casts = pd.read_pickle("datasets/casts.p")

# Preview one of the DataFrames
casts.head(3)
casts.tail(3)
```

|movie_id|cast_id|character|  gender |      id   |            name|
|---     |---     |---      |---     |---        |---              |
|5       |22      |Jezebel  |      1 |     3122  |      Sammi Davis|
| 5      | 23     |Diana     |  1    | 3123      | Amanda de Cadenet|
| 5       |24     |Athena     |  1    | 3124    | Valeria Golino    |

|movie_id|cast_id|character| gender|  id     |        name|
|---     |---    |---      |---    |---      |---         |
|433715  |    5  |   Sugar |     0 | 1734574  |Ariana Stephens| 
|433715  |    6  |   Drew  |     0 | 1734575  |  Bryson Funk |
 |459488 |     0 | Narrator|     0 | 1354401  | Tony Oppedisano|


> [!NOTE]
> What column to merge on?
> Chicago provides a list of taxicab owners and vehicles licensed to operate within the city, for public safety. Your goal is to merge two tables together. One table is called taxi_owners, with info about the taxi cab company owners, and one is called taxi_vehicles, with info about each taxi cab vehicle.

## EXERCISE 1
---

### Your first inner join

> [!TIP]
> Choose the column you would use to merge the two tables on using the .merge() method.
> You have been tasked with figuring out what the most popular types of fuel used in Chicago taxis are. To complete the analysis, you need to merge the taxi_owners and taxi_veh tables together on the vid column. You can then use the merged table along with the .value_counts() method to find the most common fuel_type.

```python
# Import pandas
import pandas as pd

# Import some of the course datasets
taxi_owners = pd.read_pickle("datasets/taxi_owners.p")
taxi_veh = pd.read_pickle("datasets/taxi_vehicles.p")

# Merge taxi_owners with taxi_veh on the column vid, and save the result to taxi_own_veh.
taxi_own_veh = taxi_owners.merge(taxi_veh, on='vid')
# Print the column names of the taxi_own_veh
print(taxi_own_veh.columns)
Index(['rid', 'vid', 'owner_x', 'address', 'zip', 'make', 'model', 'year',
       'fuel_type', 'owner_y'],
      dtype='object')

# Set the left and right table suffixes for overlapping columns of the merge to _own and _veh, respectively.
taxi_own_veh = taxi_owners.merge(taxi_veh, on='vid', suffixes=('_own', '_veh'))
# Print the column names of taxi_own_veh
print(taxi_own_veh.columns)
Index(['rid', 'vid', 'owner_own', 'address', 'zip', 'make', 'model', 'year',
       'fuel_type', 'owner_veh'],
      dtype='object')


# Select the fuel_type column from taxi_own_veh and print the value_counts() to find the most popular fuel_types used.
# Print the value_counts to find the most popular fuel_type
print(taxi_own_veh['fuel_type'].value_counts())
fuel_type
HYBRID                    2792
GASOLINE                   611
FLEX FUEL                   89
COMPRESSED NATURAL GAS      27
Name: count, dtype: int64
```

**The .merge() returned a table with the same number of rows as the original wards table. However, using the altered tables with the altered first row of the ward column, the number of returned rows was fewer. There was not a matching value in the ward column of the other table. Remember that .merge() only returns rows where the values match in both tables.**

## EXERCISE 2
---

### One-to-many merge

> [!NOTE]
> A business may have one or multiple owners. In this exercise, you will continue to gain experience with one-to-many merges by merging a table of business owners, called biz_owners, to the licenses table. Recall from the video lesson, with a one-to-many relationship, a row in the left table may be repeated if it is related to multiple rows in the right table. In this lesson, you will explore this further by finding out what is the most common business owner title. (i.e., secretary, CEO, or vice president)

```python
# Import pandas
import pandas as pd

# Import some of the course datasets
biz_owners = pd.read_pickle("datasets/business_owners.p")
licenses = pd.read_pickle("datasets/licenses.p")

# Starting with the licenses table on the left, merge it to the biz_owners table on the column account, and save the results to a variable named licenses_owners.
licenses_owners = licenses.merge(biz_owners, on='account')

# Group licenses_owners by title and count the number of accounts for each title. Save the result as counted_df
counted_df = licenses_owners.groupby('title').agg({'account':'count'})

# Sort counted_df by the number of accounts in descending order, and save this as a variable named sorted_df.
sorted_df = counted_df.sort_values('account', ascending=False)

# Use the .head() method to print the first few rows of the sorted_df.
print(sorted_df.head())
               account
title
PRESIDENT           6259
SECRETARY           5205
SOLE PROPRIETOR     1658
OTHER               1200
VICE PRESIDENT       970
```

## EXERCISE 3
---

### Total riders in a month

 > [!NOTE]
> Your goal is to find the total number of rides provided to passengers passing through the Wilson station (station_name == 'Wilson') when riding Chicago's public transportation system on weekdays (day_type == 'Weekday') in July (month == 7). Luckily, Chicago provides this detailed data, but it is in three different tables. You will work on merging these tables together to answer the question. This data is different from the business related data you have seen so far, but all the information you need to answer the question is provided. The cal table relates to ridership via year, month, and day. The ridership table relates to the stations table via station_id.

```python
# Import pandas
import pandas as pd

# Import some of the course datasets
ridership = pd.read_pickle("datasets/cta_ridership.p")
cal = pd.read_pickle("datasets/cta_calendar.p")
stations = pd.read_pickle("datasets/stations.p")

# Merge the ridership and cal tables together, starting with the ridership table on the left and save the result to the variable ridership_cal. If you code takes too long to run, your merge conditions might be incorrect.
ridership_cal = ridership.merge(cal, on=['year', 'month', 'day'])

# Extend the previous merge to three tables by also merging the stations table.
ridership_cal_stations = ridership.merge(cal, on=['year','month','day']) \
            				.merge(stations, on='station_id')
                            
# Create a variable called filter_criteria to select the appropriate rows from the merged table so that you can sum the rides column.
filter_criteria = ((ridership_cal_stations['month'] == 7) 
                   & (ridership_cal_stations['day_type'] == 'Weekday') 
                   & (ridership_cal_stations['station_name'] == 'Wilson'))

# Use .loc and the filter to select for rides
print(ridership_cal_stations.loc[filter_criteria, 'rides'].sum())
140005
```

## EXERCISE 4
---

### Three table merge

> [!NOTE]
> To solidify the concept of a three DataFrame merge, practice another exercise. A reasonable extension of our review of Chicago business data would include looking at demographics information about the neighborhoods where the businesses are. A table with the median income by zip code has been provided to you. You will merge the licenses and wards tables with this new income-by-zip-code table called zip_demo.

```python
# Import pandas
import pandas as pd

# Import some of the course datasets
licenses = pd.read_pickle("datasets/licenses.p")
zip_demo = pd.read_pickle("datasets/zip_demo.p")
wards = pd.read_pickle("datasets/ward.p")

# Starting with the licenses table, merge to it the zip_demo table on the zip column. Then merge the resulting table to the wards table on the ward column. Save result of the three merged tables to a variable named licenses_zip_ward.
licenses_zip_ward = licenses.merge(zip_demo, on='zip') \
            			.merge(wards, on='ward')
                        
# Group the results of the three merged tables by the column alderman and find the median income.
# Print the results by alderman and show median income
print(licenses_zip_ward.groupby('alderman').agg({'income':'median'}).head(10))
                        income
alderman
Ameya Pawar             66246.0
Anthony A. Beale        38206.0
Anthony V. Napolitano   82226.0
Ariel E. Reyboras       41307.0
Brendan Reilly         110215.0
Brian Hopkins           87143.0
Carlos Ramirez-Rosa     66246.0
Carrie M. Austin        38206.0
Chris Taliaferro        55566.0
Daniel "Danny" Solis    41226.0
```

## EXERCISE 4
---

### One-to-many merge with multiple tables

> [!NOTE]
> In this exercise, assume that you are looking to start a business in the city of Chicago. Your perfect idea is to start a company that uses goats to mow the lawn for other businesses. However, you have to choose a location in the city to put your goat farm. You need a location with a great deal of space and relatively few businesses and people around to avoid complaints about the smell. You will need to merge three tables to help you choose your location. The land_use table has info on the percentage of vacant land by city ward. The census table has population by ward, and the licenses table lists businesses by ward.

```python
# Import pandas
import pandas as pd

# Import some of the course datasets
licenses = pd.read_pickle("datasets/licenses.p")
land_use = pd.read_pickle("datasets/land_use.p")
census =  pd.read_pickle("datasets/census.p")

# Merge land_use and census on the ward column. Merge the result of this with licenses on the ward column, using the suffix _cen for the left table and _lic for the right table. Save this to the variable land_cen_lic.
land_cen_lic = land_use.merge(census, on='ward') \
            			.merge(licenses, on='ward', suffixes=('_cen', '_lic'))
                        
# Group land_cen_lic by ward, pop_2010 (the population in 2010), and vacant, then count the number of accounts. Save the results to pop_vac_lic.
pop_vac_lic = land_cen_lic.groupby(['ward', 'pop_2010', 'vacant'], 
                                   as_index=False).agg({'account':'count'})
                                   
# Sort pop_vac_lic by vacant, account, andpop_2010 in descending, ascending, and ascending order respectively. Save it as sorted_pop_vac_lic.
sorted_pop_vac_lic = pop_vac_lic.sort_values(['vacant', 'account', 'pop_2010'], ascending=[False, True, True])

# Print the top few rows of sorted_pop_vac_lic
print(sorted_pop_vac_lic.head())
     war  pop_2010   vacant    account
47    7     51581      19       80
12   20     52372      15      123
1    10     51535      14      130
16   24     54909      13       98
7    16     51954      13      156
```

## EXERCISE 5
---

### Counting missing rows with left join

>[!NOTE]
> The Movie Database is supported by volunteers going out into the world, collecting data, and entering it into the database. This includes financial data, such as movie budget and revenue. If you wanted to know which movies are still missing data, you could use a left join to identify them. Practice using a left join by merging the movies table and the financials table.

```python
# Import pandas
import pandas as pd

# Import some of the course datasets
movies = pd.read_pickle("datasets/movies.p")
financials = pd.read_pickle("datasets/financials.p")

# Merge the movies table, as the left table, with the financials table using a left join, and save the result to movies_financials.
movies_financials = movies.merge(financials, on='id', how='left')

# Count the number of rows in movies_financials with a null value in the budget column.
number_of_missing_fin = movies_financials['budget'].isna().sum()
# Print the number of movies missing financials
print(number_of_missing_fin)
1574
```

>[!IMPORTANT]
> If your goal is to enhance or enrich a dataset, then you do not want to lose any of your original data. A left join will do that by returning all of the rows of your left table, while using an inner join may result in lost data if it does not exist in both tables.
> A left join will return all of the rows from the left table. If those rows in the left table match multiple rows in the right table, then all of those rows will be returned. Therefore, the returned rows must be equal to if not greater than the left table. Knowing what to expect is useful in troubleshooting any suspicious merges

## EXERCISE 6
---

### Self join

> [!NOTE]
> Merging a table to itself can be useful when you want to compare values in a column to other values in the same column.

```python
# Import pandas
import pandas as pd

# Import the dataset
crews = pd.read_pickle("datasets/crews.p")

# To a variable called crews_self_merged, merge the crews table to itself on the id column using an inner join, setting the suffixes to '_dir' and '_crew' for the left and right tables respectively.
crews_self_merged = crews.merge(crews, on='id', suffixes=('_dir', '_crew'))

# Create a Boolean index, named boolean_filter, that selects rows from the left table with the job of 'Director' and avoids rows with the job of 'Director' in the right table.
boolean_filter = ((crews_self_merged['job_dir'] == 'Director') & 
     (crews_self_merged['job_crew'] != 'Director'))

direct_crews = crews_self_merged[boolean_filter]

# Print the first few rows of direct_crews
print(direct_crews.head())
        id department_dir   job_dir       name_dir department_crew        job_crew          name_crew
156  19995      Directing  Director  James Cameron         Editing          Editor  Stephen E. Rivkin
157  19995      Directing  Director  James Cameron           Sound  Sound Designer  Christopher Boyes
158  19995      Directing  Director  James Cameron      Production         Casting          Mali Finn
160  19995      Directing  Director  James Cameron         Writing          Writer      James Cameron
161  19995      Directing  Director  James Cameron             Art    Set Designer    Richard F. Mays

# pandas treats a merge of a table to itself the same as any other merge. Therefore, it does not limit you from chaining multiple .merge() methods together.

# import ratings dataset
ratings = pd.read_pickle("datasets/ratings.p")

# Merge movies and ratings on the index and save to a variable called movies_ratings, ensuring that all of the rows from the movies table are returned.
movies_ratings = movies.merge(ratings, left_index=True, right_index=True)

# Print the first few rows of movies_ratings
print(movies_ratings.head())
    id_x                 title  popularity release_date    id_y  vote_average  vote_count
0    257          Oliver Twist   20.415572   2005-09-23   19995           7.2     11800.0
1  14290  Better Luck Tomorrow    3.877036   2002-01-12     285           6.9      4500.0
2  38365             Grown Ups   38.864027   2010-06-24  206647           6.3      4466.0
3   9672              Infamous    3.680896   2006-11-16   49026           7.6      9106.0
4  12819       Alpha and Omega   12.300789   2010-09-17   49529           6.1      2124.0
```

## EXERCISE 7
---

### Do sequels earn more?

>[!NOTE]
> It is time to put together many of the aspects that you have learned in this chapter. In this exercise, you'll find out which movie sequels earned the most compared to the original movie. To answer this question, you will merge a modified version of the sequels and financials tables where their index is the movie ID. You will need to choose a merge type that will return all of the rows from the sequels table and not all the rows of financials table need to be included in the result. From there, you will join the resulting table to itself so that you can compare the revenue values of the original movie to the sequel. Next, you will calculate the difference between the two revenues and sort the resulting dataset.

```python
# Import pandas
import pandas as pd

# Import the dataset
sequels = pd.read_pickle("datasets/sequels.p")
financials = pd.read_pickle("datasets/financials.p")

# With the sequels table on the left, merge to it the financials table on index named id, ensuring that all the rows from the sequels are returned and some rows from the other table may not be returned, Save the results to sequels_fin.
sequels_fin = sequels.merge(financials, on='id', how='left')

# Merge the sequels_fin table to itself with an inner join, where the left and right tables merge on sequel and id respectively with suffixes equal to ('_org','_seq'), saving to orig_seq.
orig_seq = sequels_fin.merge(sequels_fin, how='inner', left_on='sequel', 
                             right_on='id', right_index=True,
                             suffixes=('_org','_seq'))

# Add calculation to subtract revenue_org from revenue_seq 
orig_seq['diff'] = orig_seq['revenue_seq'] - orig_seq['revenue_org']

# Select the title_org, title_seq, and diff columns of orig_seq and save this as titles_diff.
titles_diff = orig_seq[['title_org', 'title_seq', 'diff']]

# Sort by titles_diff by diff in descending order and print the first few rows.
print(titles_diff.sort_values('diff', ascending=False).head())
                                title_org                 title_seq         diff
2929                        Before Sunrise  The Amazing Spider-Man 2  700182027.0
1256   Star Trek III: The Search for Spock                The Matrix  376517383.0
293   Indiana Jones and the Temple of Doom              Man of Steel  329845518.0
1084                                   Saw          Superman Returns  287169523.0
1334                        The Terminator          Star Trek Beyond  265100616.0
```

## EXERCISE 8
---

### Performing an anti join

> [!NOTE]
> Check up right, anti and semi join technique in another resources.

