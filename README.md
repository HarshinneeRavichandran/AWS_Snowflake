# AWS_Snowflake
End to End Data Analytics Project – JSON, AWS, SNOWFLAKE, SQL 
OVERVIEW: In this project we do both sentimental analysis and general analysis as well.  We are going to ingest the JSON reviews data and the JSON business data into AWS S3 and then into snowflake and we will convert this Json into tabular format so we will create two tables one for reviews one for businesses.
Then we will create a UDF user defined functions in SQL again which can do sentimental analysis so the reviews given by the user the text will be in will be a input to this UDF and it will tell if it is a positive review a neutral review or a negative review.
Apart from that we will do data analysis using SQL for the tabular data in Snowflake.

Overview of Dataset and Project
Dataset Overview
This project utilizes two JSON datasets:
1.	Restaurant Names and Metadata - Contains details about restaurants such as ID, name, location, and cuisine type.
2.	Restaurant Reviews - Includes customer reviews, ratings, and comments associated with restaurants.
Project Goals
The objective of this project is to demonstrate an end-to-end data processing pipeline in Snowflake, covering key features such as:
•	Importing JSON Data into Snowflake
•	Transforming JSON into Structured Tables
•	Creating Zero Copy Clones and Views
•	Utilizing Time Travel for Data Recovery
•	Implementing User-Defined Functions (UDFs) for Custom Logic
________________________________________
STEP 1:
	Since the file is large, we first spilt or flatten the JSON file using Python and we load it into the S3 and then we will load it into Snowflake.
	Download the JSON dataset from KAGGLE and upload it into file folder in home directory
	Using Anaconda >  Jupiter > import the json file > check count total lines in the file > spilt the large JSON file into smaller files (10).
STEP 2:
	Create an AWS account > create an IAM account and login via IAM user 
	Create a bucket > folders > and load the JSON files

STEP 3:
	Login into Snowflake 
	Create database > create table with columns 
	Create copy into statement containing path name , AWS credentials 
	To find out AWS credentials – IAM > users > create new user > test user > attach policies (s3readonly access) > create user 
	Select user > security credentials > create access keys > other > next > create access keys > download .csv file
	Now run the snowflake queries

STEP 4: 
	Create another table and load the other file as well
	Load it in s3 first and then into snowflake 
STEP 5:
	Check the loads for both the JASON files in snowflake
	Create or replace table table_name(review_text variant); //contains only one column
Copy INTO table_name
FROM S3://path_name
CREDENTIALS = (
	AWS KEY_ID =
	AWS_SECRET_KEY = 
)
FILE_FORMAT = (TYPE = JSON)

	Convert the JSON table into columnar_table
Select review_text:column_name::datatype as edited_column_name,………..from column_name limit 10;

Steps and Explanations
Step 5: Import JSON File into Snowflake
To process JSON data, we first create tables to store it and then load the data using Snowflake's staging capabilities.
1.	Create Staging Tables
CREATE OR REPLACE TABLE restaurant_metadata (
    raw_data VARIANT
);

CREATE OR REPLACE TABLE restaurant_reviews (
    raw_data VARIANT
);
2.	Stage and Load JSON Data
CREATE OR REPLACE STAGE json_stage;
COPY INTO restaurant_metadata FROM @json_stage FILE_FORMAT = (TYPE = 'JSON');
COPY INTO restaurant_reviews FROM @json_stage FILE_FORMAT = (TYPE = 'JSON');


Step 6: Convert JSON into Structured Table
Extract useful fields from the raw JSON data stored in the VARIANT column.
CREATE OR REPLACE TABLE restaurant_details AS
SELECT 
    restaurant.value:"restaurant_id"::STRING AS restaurant_id,  
    restaurant.value:"name"::STRING AS restaurant_name,  
    restaurant.value:"cuisine"::STRING AS cuisine,  
    restaurant.value:"cost"::STRING AS cost  
FROM restaurant_metadata,  
LATERAL FLATTEN(input => raw_data) AS restaurant;


LATERAL is a table function that helps when dealing with nested structures, especially JSON arrays. It allows each row in the main table to reference the result of a subquery or table function applied to that row.
•	It helps expand JSON arrays into separate rows.
•	It allows referencing columns from the main query inside a subquery.
•	It is commonly used with FLATTEN(), which extracts values from arrays.



Step 7: Zero Copy Clone & Views
Zero Copy Clone
Creates a duplicate table instantly without extra storage costs.
CREATE OR REPLACE TABLE restaurant_details_clone CLONE restaurant_details;
CREATE OR REPLACE TABLE restaurant_reviews_clone CLONE restaurant_reviews_table;
Views
A view provides a virtual representation of data without duplicating it.
CREATE OR REPLACE VIEW restaurant_info_view AS
SELECT
    d.restaurant_id,
    d.name,
    d.location,
    d.cuisine,
    r.rating,
    r.comment
FROM restaurant_details d
LEFT JOIN restaurant_reviews_table r ON d.restaurant_id = r.restaurant_id;
Step 8: Time Travel in Snowflake
Allows recovery of deleted or modified data within a specific retention period.
Simulate Deletion
DELETE FROM restaurant_details WHERE name = 'Some Restaurant';
Restore Data using Time Travel
UNDROP TABLE restaurant_details;
OR
CREATE OR REPLACE TABLE restaurant_details_recovered AS
SELECT * FROM restaurant_details AT (OFFSET => -60*5);

Step 9: User-Defined Function (UDF)
UDFs allow custom transformations on data.
Creating a UDF to Classify Ratings
CREATE OR REPLACE FUNCTION classify_rating(rating FLOAT)
RETURNS STRING
LANGUAGE SQL
AS
$$
CASE
    WHEN rating >= 4.5 THEN 'Excellent'
    WHEN rating >= 3.5 THEN 'Good'
    WHEN rating >= 2.5 THEN 'Average'
    ELSE 'Poor'
END
$$;
Using the UDF in Queries
SELECT
    review_id,
    rating,
    classify_rating(rating) AS rating_category
FROM restaurant_reviews_table;
________________________________________
3. Explanation of Key Concepts
Time Travel
•	Snowflake’s Time Travel feature allows users to access historical data.
•	It helps recover from accidental deletions or modifications.
•	Syntax: SELECT * FROM table_name AT (OFFSET => -60*5);
Zero Copy Clone
•	Creates an exact duplicate of a table, schema, or database without consuming additional storage.
•	Any new changes to the cloned table are independent of the original table.
•	Syntax: CREATE TABLE new_table CLONE existing_table;
Views
•	A View is a virtual table that stores no data but represents a query result.
•	Useful for simplifying complex queries and securing data access.
•	Syntax: CREATE VIEW view_name AS SELECT * FROM table_name;
User-Defined Functions (UDFs)
•	UDFs allow for custom processing of data.
•	Snowflake supports SQL-based and JavaScript-based UDFs.
•	Example: Classifying ratings into categories.
________________________________________
Conclusion
✅ Successfully imported and processed JSON files in Snowflake.
✅ Transformed JSON into structured tables.
✅ Used Zero Copy Clone to create table duplicates without extra storage.
✅ Created Views for simplified querying.
✅ Implemented Time Travel to recover lost data.
✅ Used UDFs to classify restaurant ratings dynamically.



