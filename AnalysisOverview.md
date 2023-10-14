# <img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/1e7a079a-b98a-44e5-83e1-5777c2f23915" width = "100"> 
# Understanding Consumer Cancellation Habits: Exploratory Data Analysis Using Python
### Author: Dylan Viyar 

### Last Updated: October 17, 2023

Dylan Viyar's Cymbiotika technical assessment. 

# 1. Background Information

### 1.0 The Setting

In an effort to understand trends in user cancellations, we can analyze the various tiers of users at the time of cancellation to determine if there is any correlation between the tier a user was in and the reason for cancellation.

### 1.1 Guiding Questions:

1. What tier was each customer in at the time of their cancellation?
2. What are some trends in consumer cancellations? (Common cancellation times, length of between cancellations and tier aquisition etc.)
3. How understanding these trends help Cymbiotika make smarter and more consumer-oriented business decisions?

### 1.2 The Business Task:

*Analyze the provided JSON data to determine the total number of canceled customers per tier, provide a csv file of the customers with the correct tier assigned and a copy of your Python script that you used to perform the data manipulation*

# 2. The Data

### 2.1 Information about the Datasets

1. There are two provided files, `Customer_tiers.json` and `Cancellation_data.json` with information about customer tier aquisition and customer cancellation data, respectively
2. There will be customers with multiple cancellation dates with the dataset
3. Tiers ascend 1-4
4. Tier data will contain promotions and demotions per Customer
5. The data is stored in JSON data types

# 3. Data Preparation

#### Before beginning analysis it is crucial to ensure the dataset is relevant, complete and committed to answering our business task.

### 3.1 Cleaning our Data

Some **Key Steps** include:
1. Dealing with NULL and missing values
2. Removing duplicates
3. Checking for data type errors (inconsistent/mismatched data types)
4. Ensuring the data stays relevant to the business task

To get a basic information of the datasets we can use the .info() method:

```python
import pandas as pd
import json
from datetime import datetime

# Load JSON data into Pandas DataFrames

with open("Customer_tiers.json", "r") as customer_file:
    customer_tiers_data = json.load(customer_file)

with open("Cancellation_data.json", "r") as cancellation_file:
    cancellation_data = json.load(cancellation_file)

customer_tiers_df = pd.DataFrame(customer_tiers_data)
cancellation_df = pd.DataFrame(cancellation_data)
print(customer_tiers_df.info())
print(cancellation_df.info())
```
<img src ="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/cdd8ea5e-86fc-49ec-82c2-5ab3cab35b18" width = "300">


It seems to be the case that there are no null values, however it is best practice to double-check and also ensure that there are no duplicate values that skew our data.

In order to check for duplicate rows, we can utilize the `duplicated()` methos in the pandas library:

```python
# Data Cleaning
# Checking for duplicate rows
tiers_duplicates = customer_tiers_df[customer_tiers_df.duplicated(keep='first')]
print("Number of Duplicated rows in Tiers Data:")
print(tiers_duplicates)

cancellation_duplicates = cancellation_df[cancellation_df.duplicated(keep='first')]
print("Number of Duplicated rows in Cancellation Data:")
print(cancellation_duplicates)
```

<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/9227abb8-606c-495c-861e-9793b381dac8" width = "500">

We can see that there are no duplicated rows in our dataset that we have to deal with.

Similarly we can look for null values with the isnull() method:

```python
# Checking for Null Values

tiers_null = customer_tiers_df.isnull()
tiers_null_counts = tiers_null.sum()

print("Number of null values in each column:")
print(tiers_null_counts)

cancellation_null = cancellation_df.isnull()
cancellation_null_counts = cancellation_null.sum()

print("Number of null values in each column:")
print(cancellation_null_counts)
```

<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/81e24234-fa92-417d-bbdd-e18ddd9ddb1d" width= "250">

No null values to deal with either! Nice!

Our data seems to be clean and ready for exploration. We can proceed to data analysis.

# 4. Data Analysis

### 4.0 Environment Setup

For our project, we can utilize the json and pandas library to manipulate our data. We also set up datetime to deal with the date values in our dataframe:

```python
import pandas as pd
import json
from datetime import datetime
```
### 4.1 Code Walkthrough

```python
# Load JSON data into Pandas DataFrames

with open("customer_tiers.json", "r") as customer_file:
    customer_tiers_data = json.load(customer_file)

with open("cancellation_data.json", "r") as cancellation_file:
    cancellation_data = json.load(cancellation_file)

customer_tiers_df = pd.DataFrame(customer_tiers_data)
cancellation_df = pd.DataFrame(cancellation_data)
```

We can cast the JSON data into panda dataframes using the pd.DataFrame function.


