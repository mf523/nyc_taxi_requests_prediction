# NYC Taxi Requests Prediction

ABC company is a ride hailing company, they have large volume of subscribe users using their mobile app to get transportation services from local drivers. The mobile app for passengers and drivers will upload activities data to server for data analyst. ABC company wants to leverage AI/Machine Learning technologies to improve their business. One of their key requirements is demand forecast. They prefer to split a city into different grid, and forecast the demand in each grid at 30min slot. If the demand goes high in future, ABC company will increase the price in that grid to slow down the demands.

To demonstrate the demand forecast in each grid at 30min slot, we use the yellow New York City Taxi and Limousine Commission (TLC) Trip Record Data between April 2018 and June 2018 in Manhattan from AWS public datasets as source data (https://registry.opendata.aws/nyc-tlc-trip-records-pds/). We split the dataset into train part (2018.04.01-2018.05.31) and validate part (2018.06.01-2018.06.30). We demonstrate 3 methods to forecast demand: XGBoost, LightGBM, linear regression implemented using sklearn, and evaluate the models using mean absolute error (MAE).

We mainly use linear regression algorithm as the baseline algorithm, and use XGBoost as the main algorithm. XGBoost is an optimized distributed gradient boosting library designed to be highly efficient, flexible and portable. It implements machine learning algorithms under the Gradient Boosting framework. XGBoost provides a parallel tree boosting (also known as GBDT, GBM) that solve many data science problems in a fast and accurate way.

To run this notebook, you have to download trip data from Amazon S3 bucket to nyc_tlc directory in your computer:

## [Question 1]

Download data and unzip archive file commands

<pre>
https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2018-04.csv
https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2018-05.csv
https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2018-06.csv
https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
https://s3.amazonaws.com/nyc-tlc/misc/taxi_zones.zip (unzip)
</pre>

After download, your local directory structure should be:

<pre>
nyc_tlc
├── misc
│   ├── taxi_zone_lookup.csv
│   ├── taxi_zones
│   │   ├── taxi_zones.dbf
│   │   ├── taxi_zones.prj
│   │   ├── taxi_zones.sbn
│   │   ├── taxi_zones.sbx
│   │   ├── taxi_zones.shp
│   │   ├── taxi_zones.shp.xml
│   │   └── taxi_zones.shx
│   └── taxi_zones.zip
└── trip_data
    ├── yellow_tripdata_2018-04.csv
    ├── yellow_tripdata_2018-05.csv
    └── yellow_tripdata_2018-06.csv

3 directories, 12 files
</pre>

### Basic Prepare

We import all useful packages, and set the `first_datetime` to 2018-04-01 00:00:00, and `last_datetime` to 2018-07-01 00:00:00. We split the dataset into two parts: train and validate, by setting the `train_valid_split_datetime` to 2018-06-01 00:00:00.

### Taxi Zones

Since newest NYC Taxi dataset only provides `PULocationID` and `DOLocationID`, instead of `pickup_longitude`, `pickup_latitude`, `dropoff_longitude`, and `dropoff_latitude` in previous dataset, we can only predict requests in each `PULocationID` (zone). We load [taxi _zone_lookup.csv] and [taxi_zones.shp], and use `geopandas` to visualize the zones in Manhattan (69 in total).

### Data Prepare

We load all data from [nyc_tlc/trip_data/] between April and June 2018, and filter abnormal data. We use `matplotlib` and `geopandas` to visualize some columns and help us to understand the trip data.

## [Question 2]

1. load Manhattan data: from 2018-04 to 2018-06
2. define a function 'filter_abnormal_data' to filter abnormal data
3. call filter_abnormal_data to filter 'contest_helper.NycTaxiAnalyzer.data'

## [Question 3]

Show statistics of the prepared sample data. 

## [Challenge Question]

Add new prediction algorithm.

### Feature Prepare

We set the `30min_id` to represent 30min slot. For example, time between 2018-01-01 00:00:00 and 2018-01-01 00:05:00 has a `5min_id` as 0, and time between 2018-01-01 00:05:00 and 2018-01-01 00:10:00 has a `5min_id` as 1, and the similar with `15min_id` and `30min_id`. For each `Xmin_id` (X represents 5, 15 or 30), we predict the requests in all 69 zones. We have some `static features` such as `month`, `day`, `hour`, `weekday`, `is_weekend`, `is_morning_peak`, `is_evening_pick` for all `Xmin_id` and zones. Also we can extend more static features such as weather and zone features. Other `dynamic features` includes requests in `5min ago`, `10min ago`, `15min ago`, `7days ago`, etc. Also we can extend more dynamic features such as total passengers in 5min ago. At last, we generate 34 features for each `Xmin_id` and zone.

### Train and Validate

We split all data into train and validate part. We demonstrate 3 methods to forecast requests: XGBoost, LightGBM, linear regression implemented using sklearn, and evaluate the models using mean absolute error (MAE).