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

```python
# Load JSON data into Pandas DataFrames

with open("customer_tiers.json", "r") as customer_file:
    customer_tiers_data = json.load(customer_file)

with open("cancellation_data.json", "r") as cancellation_file:
    cancellation_data = json.load(cancellation_file)

customer_tiers_df = pd.DataFrame(customer_tiers_data)
cancellation_df = pd.DataFrame(cancellation_data)

#Create datetime objects

customer_tiers_df['date_earned'] = pd.to_datetime(customer_tiers_df['date_earned'])
cancellation_df['churn_date'] = pd.to_datetime(cancellation_df['churn_date'])
```

Here, we are reading in the JSON files and turning those files into panda dataframes. We then cast the columns relevant that contain date values into datetime to make it easier for calculations later on in the analysis.

```python
#Merge cancellation_df with the customer_tier_df with a left merge on the external_customer_id

merged_df = pd.merge(cancellation_df, customer_tiers_df, on='external_customer_id', how='left')
```
This is a crucial step in the code. We want to merge the two dataframes so that we can gain information from both sources in one singular dataframe. Knowing that we need all of the cancellation data (we are counting the total number of cancellations per tier) we merge from the left, with cancellation_df as the left dataframe. This ensures that we have all the cancellation data and the respective customer tier data that corresponds with all the cancellation data. We can utilize the unique identifier `external_customer_id` that is present in both dataframes to merge the two together.

```python
filtered_df = merged_df[merged_df['date_earned'] <= merged_df['churn_date']]

#Group filtered_df and determine tier closest in time to the churn date

result_df = filtered_df.groupby(['churn_date', 'external_customer_id']).apply(lambda x: x.loc[(x['date_earned'] - x['churn_date']).abs().idxmin()])
```
Here, we filter `merged_df` to ensure that there are no rows such that the date in which a customer left is less than the date in which a customer earned a tier. (Such values are irrelevant to the business task.) We then utilize the `groupby()` function to group by the `churn_date` and the `external_customer_id`, effectively creating 
groups for unique combinations of `churn_date` and `external_customer_id`. We then apply the lambda function to specify a custom operation that is applied to each group of data. The operation we apply finds the smallest difference of the `date_earned` minus the `churn_date`. `idxmin()` then finds the index of the row within each grouping where the difference is the smallest. Finally, `x.loc` is used to locate the row with the smallest time difference within each group and select that row, getting a new dataframe with only the rows that have the smallest time difference between the `date_earned` and the `churn_date`. In the context of the business task, this code creates a dataframe in which all the rows have the tier when the customer cancelled.

```python
result_df = result_df[['churn_date', 'external_customer_id', 'tier_id', 'tier']]

canceled_customers_per_tier = result_df['tier'].value_counts()

print("Number of canceled customers per tier:")
print(canceled_customers_per_tier)

result_df.to_csv("tiers_of_canceled_customers.csv", index=False)
```
Here, we reconfigure `result_df` to have the columns that are desired, then we utilize the `value_counts()` methos to aggregate the total number of customers per tier, then print the aggregation and save the required dataframe into a CSV file.


After running our code, the output is the curation of the `tiers_of_canceled_customers.csv` CSV file and the following dataframe:

<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/550c028f-9b82-401b-8683-b722455bdba1" width= "300">

We see that cancellation tier distribution is the following:
- Tier #1: 5852 cancellations
- Tier #2: 843 cancellations
- Tier #3: 1104 cancellations
- Tier #4: 1633 cancellations

### 4.2 Additional Analysis

With the provided datasets, one can conduct even further analysis. A natural question is, *When are cancellations most common?* The following process demonstrates how one can find the answer:

```python
#Adding column with just the month name from the date

result_df['month_name'] = result_df['churn_date'].dt.strftime('%B')

# Group by the 'month_name' column and count cancellations in each month
cancellations_per_month = result_df.groupby('month_name')['external_customer_id'].count().reset_index()

# Rename the columns for clarity
cancellations_per_month.columns = ['Month', 'Number_of_Cancellations']

print(cancellations_per_month)
```
<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/fb2ad801-0815-4883-b40e-94ca67304b44" width="250">

One can see that there are more cancellations in August in comparison to September, and if provided more data, one can determine annual trends, if any, in consumer cancellations.

# 5. Data Visualization

Business Intelligence visualization tools such as Tableau can help us easily understand our findings and better communicate analysis with stakeholders. 

![CancellationsPerTierGraph](https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/f2a547a0-6880-4f6f-8774-a0e35251ebd1)



<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/6fc5e4f1-8749-4a79-8c7b-7c70f92889c6" width="200">

Utilitizing simple bar graphs, it is easier to understand the habits of our data. The cancellation amount for tier 1 is drastically larger than the rest of the tiers, as seen in the first visualization. We can also see that the amount of cancellations in August are slightly larger than the cancellations in September.

# 6. Conclusion

### 6.1 Key Takeaways

- Tier #1 has the most cancellations
- August had more cancellation than September
- Lowest amount of cancellations were in Tier #2 possibly due to customers not wanting to cancel in hope of promotion
- Tier #4 has the second highest number of cancellations, possibily because customer's understand that there is no higher tier

### 6.2 Next Steps

Further related analysis can be conducted upon retrieving more data. Some examples of potential additional data exploration include:

1. Yearly Analysis
   - Determine annual trends of customer tier history and determine if there are certain seasonal patterns to note
2. Tier History Analysis
   - Analyze how customer's get promoted/demoted, and various factors that contribute to customer tier status
3. Geographic Analysis
   - Determine if a customer's location has an effect on customer retention rate

### Thank you!

Thank you for your time to understand my data analysis and process. Please feel free to reach out with any questions or concerns!
   


