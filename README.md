#### Project Summary
Immigration data and city temperature data provided by Udacity is going to be used to create a star schema that allow users to see if there is a relation between City Temperature and immigration requests to this City.


### Step 1: Scope the Project and Gather Data
#### Scope
The scope of the project is to provide an analytical schema for users using provided immigration dataset and temperature dataset so user can use the generated model to find answer for questions like is there a relationship between city temperature and number of immigrant? Do people from countries with warmer or cold climate immigrate to the US in large numbers?

The country dimension table is made up of data from the global land temperatures by city and the immigration datasets. The combination of these two datasets allows analysts to study correlations between global land temperatures and immigration patterns to the US and get insights into migration patterns into the US based on the temperature, demographics as well as overall population of states. We could also ask another questions such as, do populous states attract more visitors on a monthly basis?

#### Describe and Gather Data

**I94 Immigration Data:**
This dataset comes from the US National Tourism and Trade Office. it is provided as SAS files by Udacity.
main fields in the dataset are
i94yr : 4 digit year
i94mon : numeric month
i94cit : code of origin city
i94port : code of destination city
arrdate : arrival date
i94mode : travel code
depdate : departure date
i94visa : Visa codes collapsed into three categories Business, Pleasure and Student
i94bir : Age of Respondent in Years

**World Temperature Data:**
This dataset came from Kaggle and provided as CSV file by Udacity. this dataset has the follwoing fields AverageTemperature, City, Country, Latitude and Longitude

### Step 2: Explore and Assess the Data
I used Panda DataFrame to explore the data after loading it from files examples are
display the first 5 rows of the data (df.head)
check count of rows(df.shap)
display columns(df.columns)
get statistics about data (df.describe())
get max and min values for avgerage temperature column to see if there is an outliers
check unique values of some columns


### Step 3: Define the Data Model

- 3.1 Conceptual Data Model
we will create a star schema with the two dimensions tables and on fact table because the star schema is widely used and easy to understand by users

#### Dimension Tables

**Demographic_Dim:** has the following columns and populated from the I94 immigration data set.
I94YR : year
I94MON : month
I94CIT : origion city code
I94PORT : destionation city code
I94MODE : travel code
I94BIR : Age of Respondent in Years between 1 and 116
ARRDATE : Arrival date
DEPDATE : Departure date
I94VISA : Visa type (Business/Pleasure/Student)

**Temperature_Dim:** has the following columns and populated from the temperature dataset.
AverageTemperature : Average Temperature
City : City Name
Country : Country Name
Latitude : Latitude
Longitude : Longitude
I94PORT : city code mapped from valid codes in SAS Description File

**Immigration_Fact:** has the following columns populated from immigration data set and temperature dataset
year : year (I94YR)
month : month (I94MON)
origion_city : origion city code (I94CIT)
destionation_city : destionation city code (I94PORT)
travel_code : travel code (I94MODE)
respondent_age : Age of Respondent in Years between 1 and 116 (I94BIR)
arrival_date : Arrival date (ARRDATE)
departure_date : Departure date (DEPDATE)
visa_type : Visa type (Business/Pleasure/Student) (I94VISA)
avgTemp : Average Temperature

- 3.2 Mapping Out Data Pipelines
load immigration sas files and clean it using function get_Immigration_Spark_df_from_SAS_file wich return clean and ready to use spark DataFrame

- write this spark dataframe as parquet file partitiond by i9port(destination city)

- load Temperature CSV file and clean it using function get_Temperature_spark_df_from_CSV_file wich return clean and ready to use spark DataFram

- write this spark dataframe as parquet file partitiond by i9port(destination city)

- create fact table by joining the above two DataFrames and write to parquet file partition by i94port(destination city)



### Step 4: Run Pipelines to Model the Data
**4.1 Create the data model**
Build the data pipelines to create the data model by do the following
- 1- run get_Immigration_Spark_df_from_SAS_file function with immigration SAS file as an argument, this
will return clean and ready to use spark DataFrame with immigration Data.
- 2- write the returned DataFrame to Parquet file partitioned by destination city (i94port column)

- 3- run get_Temperature_spark_df_from_CSV_file function with Temerature CSV File as an argument, this
will return clean and ready to use spark DataFrame with Temperature Data.
- 4- write the returned DataFrame to Parquet file partitioned by destination city (i94port column)

- 5- create two spark views over two dataframes already created
- 6- using sparksql to join the two view to create fact DataFrame
- 7- write Fact DataFrame as parquet file  partitioned by destination city (destination_city column)

**4.2 Data Quality Checks**
i built two quality checks one check for count to see if the DataFrame is empty or not you can execute it by running
count_quality_check function with DataFrame as argument.
the second qulity check see if there is refrential integrity viloations between immigration and temperature data based on
i94port column(destination city)

**4.3 Data dictionary**

#### Dimension Tables

**Demographic_Dim:** has the following columns and populated from the I94 immigration data set.
I94YR : 4 digits year
I94MON : numeric month
I94CIT : origion city code
I94PORT : destionation city code
I94MODE : travel code
I94BIR : Age of Respondent in Years between 1 and 116
ARRDATE : Arrival date
DEPDATE : Departure date
I94VISA : Visa type (Business/Pleasure/Student)

**Temperature_Dim:** has the following columns and populated from the temperature dataset.
AverageTemperature : Average Temperature
City : City Name
Country : Country Name
Latitude : Latitude
Longitude : Longitude
I94PORT : destionation city code mapped from valid codes in SAS Description File.

Immigration_Fact: has the following columns populated by joining the two dim tables
year : year (I94YR)
month : month (I94MON)
origion_city : origion city code (I94CIT)
destionation_city : destionation city code (I94PORT)
travel_code : travel code (I94MODE)
respondent_age : Age of Respondent in Years between 1 and 116 (I94BIR)
arrival_date : Arrival date (ARRDATE)
departure_date : Departure date (DEPDATE)
visa_type : Visa type (Business/Pleasure/Student) (I94VISA)
avgTemp : Average Temperature


### Step 5: Complete Project Write Up

- **Clearly state the rationale for the choice of tools and technologies for the project**.
For exploring the data I used panda library as it is easy to use, then for processing data I preferred to use Spark as it can handle many files with large amount of data and varies data formats. Also spark sql used to perform standard SQL like operations on the data like joining the two dataframes to generate the fact table.

- We used star schema model not only it is the most common modeling paradigm but because it tends to be better for performance and it has a small number of tables and clear join paths, queries run faster than they do against an OLTP system. Small single-table queries, usually of dimension tables, are almost instantaneous.

- **Propose how often the data should be updated and why.**
since the data arrive monthly in files so we do not have the complexity of defining delta so we can load data every month when the data files arrive to our side and append to the previous data

##### Write a description of how you would approach the problem differently under the following scenarios:

- **The data was increased by 100x**.
we can increase the work power of the spark cluster (maybe using autoscaling service here will be very good) or we can change to Amazon Redshift DWH as it is scalable and can handle large amount of data

- **The data populates a dashboard that must be updated on a daily basis by 7am every day**.
we can use a tool like Airflow to schedule the batch job to run daily and configure the success and fail scenario

- **The database needed to be accessed by 100+ people.**
with this new settings we can go for Redshift Database as it is very scalable and can handle this number of requests.
