# House Prices - Advanced Regression Techniques

## Objective 

- Based on a dataset with 79 variables predict the sales price for each house. For each Id in the test set, predict the value of the SalePrice variable. 

## Data

- The data used in this project is from  [House Prices - Advanced Regression Techniques]
- This version of the dataset was downloaded from [Kaggle](https://www.kaggle.com/competitions/house-prices-advanced-regression-techniques/data)
- These are the tables that comprise the Open Dataset:
    + `yelp_academic_dataset_business.json`: Contains business data including location data, attributes, and categories.
    + `yelp_academic_dataset_checkin.json`: Checkins on a business.
    + `yelp_academic_dataset_review.json`: Contains full review text data including the user_id that wrote the review and the business_id the review is written for.
    + `yelp_academic_dataset_tip.json`: Tips written by a user on a business. Tips are shorter than reviews and tend to convey quick suggestions.
    + `yelp_academic_dataset_user.json`: User data including the user's friend mapping and all the metadata associated with the user.

## Steps

- `S01_etl.ipynb`: Extract, transform and load Yelp datasets.
    + Here `json` files are read and converted to parquet files. Data is partitioned by metropolitan_area and year to optimize Spark performance.
    + Data quality checks are perfomed and data is cleaned. Text variables are converted to lower case, json lists are unnested in yelp_academic_dataset_business. Date and time variables are extracted from timestamp.
    + Business metropolitan areas a created using KMeans algorithm, and a catalog is created
    + Cleaned datasets are stored in output for further analysis.
- `S02_eda.ipynb`: A brief exploratory analysis is conducted on the partitioned parquets and cleaned data to learn more about the data, aggregations and quality issues.
- `S03_data_model.ipynb`: The data model is implemented the dimensions and a restaurant fact table is created.
    + Create number of business checkins table
    + Create number of business tips table
    + Create number of business reviews table
    + Create business union table: Here the fact table is built for all business.
    + Create a Restaurants Catalog: A catalog for all restaurant business is created, labelling observations manually.
    + Create restaurants table. The final fact table is stored in a dataset in the following address `data/output/restaurants.csv`. 
- `S04_analytics.Rmd`: This is the analytics code where the top 10 most reviewed restaurants for each metropolitan is generated. The final report can be consulted in `S04_analytics.html`

## Repository structure

`config`: Configuration files for data and aws services.
`data`: The inputs, preprocessing and outputs are stored here.
    `input`: Here is where the original `json` files are stored for further preprocessing.
    `preprocessing`: Intermediate tables such as catalogs are stored here.
    `output`: Cleaned data are stored here as parquet files.

## Data Model
 
The data model for this project is explained in the following diagram.

![Data Model](figs/data_model.png)

Data dictionary for `restaurants.csv`

- `categories`: character. Restaurants category.
- `cluster`: double. Id metropolitan area.
- `metropolitan_area`: character. Name of metropolitan area.
- `business_id`: character. Business id.
- `name`: character. Name of business.
- `latitude`: double. Latitude.
- `longitude`: double. Longitude.
- `review_count`: double. Number of reviews that the restaurant has.
- `stars`: double. Average stars that the restaurant has.
- `year`: double. Year of the data.
- `num_checkins`: double. Number of checkins for the given year.
- `num_tips`: double. Number of tips that the users have written given the year.
- `num_reviews`: double. Number of user reviews in the dataset for this business in the given year.
- `mean_stars_reviews`: double. Number of mean stars for the user reviews.
- `is_restrautrant`: logical. Is this business a restaurant.


## Technology used in current scenario

- `Spark`: Spark was used in the local environment, as the data is not big enough to demand a cluster. However, data is big enough that file partitioning optimizations had to be performed, to increase the performance of the ETL.

- `Python`: Python was used to perform the ETLs and to connect to Spark. This pipeline was implemented in Python so it could be run in a AWS EMR cluster as well.

- `R`: An R Markdown was used to analyze the data and produce the maps in leaflet.

## Production scenario

- Assuming that Yelp data can be extracted periodically, this pipeline can be executed in AWS services using a EMR cluster with Spark.

- Data should be stored in the data lake in S3, using the code `S00_stage_data.ipynb`.

- The ETL in `S01_etl.ipynb` already has the code to launch the EMR cluster, and in `config` the configuration files to setup the credentials and variables for the infrastructure will be stored.

- Based on the objective of this analysis, the pipeline could be scheduled to run with Airflow every start of a new month.
