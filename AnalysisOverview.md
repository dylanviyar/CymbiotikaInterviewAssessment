<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/454a3ab8-22b5-4413-a2f9-4a8f2392f177" width ="100">

# Understanding Consumer Cancellation Habits: Exploratory Data Analysis Using Python
### Author: Dylan Viyar 

### Last Updated: October 16, 2023

Dylan Viyar's Cymbiotika technical assessment. 

# 1. Background Information

### 1.0 The Setting

In an effort to understand trends in user cancellations, we can analyze the various tiers of users at the time of cancellation to determine if there is any correlation between the tier a user was in and the reason for cancellation.

### 1.1 Guiding Questions:

1. How many cancellations occurred per tier?
2. What are some trends in consumer cancellations? 
3. How can understanding these trends help Cymbiotika make smarter and more consumer-oriented business decisions?

### 1.2 The Business Task:

*Analyze the provided JSON data to determine the total number of canceled customers per tier, provide a csv file of the customers with the correct tier assigned and a copy of your Python script that you used to perform the data manipulation*

# 2. The Data

### 2.1 Information about the Datasets

1. There are two provided files, `Customer_tiers.json` and `Cancellation_data.json` with information about customer tier aquisition and customer cancellation data, respectively
2. There will be customers with multiple cancellation dates with the dataset
3. Tiers ascend 1-4
4. Tier data will contain promotions and demotions per customer
5. The data is stored in JSON data types

# 3. Data Preparation

#### Before beginning analysis, it is crucial to ensure the dataset is relevant, complete and committed to answering our business task.

### 3.1 Cleaning our Data

Some **Key Steps** include:
1. Dealing with NULL and missing values
2. Removing duplicates
3. Checking for data type errors (inconsistent/mismatched data types)
4. Ensuring the data stays relevant to the business task

To get a basic information of the datasets we can use the .info() method in the python library pandas:

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
<img src ="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/b8c8b53e-fcb2-497a-b755-6c3bdf557cb5" width="300">

It seems to be the case that there are no null values, however it is best practice to double-check and also ensure that there are no duplicate values that skew our data.

In order to check for duplicate rows, we can utilize the `duplicated()` method in the pandas library:

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
<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/05af15b8-dd94-4d64-a4db-ae6cf9724812" width = "500">

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
<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/30a9476d-f2a8-4643-ab62-70c6d8fab42d" width="250">

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

#Create datetime objects

customer_tiers_df['date_earned'] = pd.to_datetime(customer_tiers_df['date_earned'])
cancellation_df['churn_date'] = pd.to_datetime(cancellation_df['churn_date'])
```

To begin, we are reading in the JSON files and turning those files into panda dataframes. We then cast the relevant columns that contain date values into datetime to make it easier for calculations later on in the analysis.

```python
#Merge cancellation_df with the customer_tier_df with a left merge on the external_customer_id

merged_df = pd.merge(cancellation_df, customer_tiers_df, on='external_customer_id', how='left')
```
This is a crucial step in the code. We want to merge the two dataframes so that we can gain information from both sources in one singular dataframe. Since we need all of the cancellation data (we are counting the total number of cancellations per tier), we merge from the left, with `cancellation_df` as the left dataframe. This ensures that we have all the cancellation data and the respective customer tier data that corresponds with it. We can utilize the unique identifier `external_customer_id` that is present in both dataframes to merge the two together.

```python
filtered_df = merged_df[merged_df['date_earned'] <= merged_df['churn_date']]

#Group filtered_df and determine tier closest in time to the churn date

result_df = filtered_df.groupby(['churn_date', 'external_customer_id']).apply(lambda x: x.loc[(x['date_earned'] - x['churn_date']).abs().idxmin()])
```
Here, we filter `merged_df` to ensure that there are no rows such that the date in which a customer canceled is less than the date in which a customer earned a tier. (Such values are irrelevant to the business task.) We then utilize the `groupby()` function to group by the `churn_date` and the `external_customer_id`, effectively creating 
groups for unique combinations of `churn_date` and `external_customer_id`. Then, we apply the lambda function to specify a custom operation that is applied to each group of data. The operation we apply finds the smallest difference of the absolute value of `date_earned` minus the `churn_date`. `idxmin()` then finds the index of the row within each grouping where the difference is the smallest. Finally, `x.loc` is used to locate the row with the smallest time difference within each group and select that row, obtaining a new dataframe with only the rows that have the smallest time difference between the `date_earned` and the `churn_date`. In the context of the business task, this code snippet creates a dataframe in which all the rows have the customer's tier when they canceled.

```python
result_df = result_df[['churn_date', 'external_customer_id', 'tier_id', 'tier']]

canceled_customers_per_tier = result_df['tier'].value_counts()

print("Number of canceled customers per tier:")
print(canceled_customers_per_tier)

result_df.to_csv("tiers_of_canceled_customers.csv", index=False)
```
Here, we reconfigure `result_df` to have the desired columns. Then, we utilize the `value_counts()` method to aggregate the total number of customer cancellations per tier. Finally, we print the aggregation and save the required dataframe into a CSV file.

After running our code, the output is the curation of the `tiers_of_canceled_customers.csv` CSV file and the following dataframe:

<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/937a9fea-2eef-45d4-be5c-3da2c6e32a61" width="300">

We see that the cancellation per tier distribution is the following:
- Tier #1: 5852 cancellations
- Tier #2: 843 cancellations
- Tier #3: 1104 cancellations
- Tier #4: 1633 cancellations

### 4.2 Additional Analysis

With the provided datasets, we can conduct even further analysis. A natural question is *"When are cancellations most common?"* The following process demonstrates how we can find the answer:

```python
# Finding cancellations per month
#Adding column with just the month name from the date

result_df['month_name'] = result_df['churn_date'].dt.strftime('%B')

# Group by the 'month_name' column and count cancellations in each month
cancellations_per_month = result_df.groupby('month_name')['external_customer_id'].count().reset_index()

# Rename the columns for clarity
cancellations_per_month.columns = ['Month', 'Number_of_Cancellations']

print(cancellations_per_month)
```
<img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/717c03fc-a23f-40c3-a7f8-5725149b02bb" width="250">

We can see that there are more cancellations in August in comparison to September. If provided more data, the above code can also determine annual trends, if any, in consumer cancellations.

Another avenue of analysis explores the *average time it takes to cancel after tier aquisition for each tier.* This form of analysis can supplement exploration in customer retention habits and trends. The following code can perform the analysis:

```python
# Finding average days to cancel per tier:
# Calculate the time difference between achieving a tier and canceling

filtered_df['time_to_cancel'] = (filtered_df['churn_date'] - filtered_df['date_earned']).dt.total_seconds()

# Group filtered_df by tier and calculate the average time to cancel in days for each tier
average_time_to_cancel_days = (filtered_df.groupby('tier')['time_to_cancel'].mean() / 86400.0).astype(int)  # 86400 seconds in a day

print("Average time to cancel (in days) per tier:")
print(average_time_to_cancel_days)
```
<img src ="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/9277ae52-8973-40ee-96af-ee04479fdbb3" width="300">

We see that Tier #1 has the fastest average time to cancel whereas Tier #2-4 are very close in average cancellation turnaround times.

# 5. Data Visualization

Business intelligence visualization tools such as Tableau can help us easily understand our findings and better communicate analysis with stakeholders. The following graphs were curated with Tableau to view our data.

![CancellationsPerTierGraph](https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/3d7fb69e-9243-4de9-94fa-105601040afa)

<img src ="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/c8b124c1-41ea-443e-9dc3-92c970059c4f" width="200"> <img src="https://github.com/dylanviyar/CymbiotikaInterviewAssessment/assets/81194849/8aa5a757-38b9-4b91-9760-3fbad68621ae" width="600">

Utilitizing simple bar graphs, it is easier to visualize the habits of our data. 

**Notable trends:**
- The cancellation amount for Tier #1 is drastically larger than the rest of the tiers
- The amount of cancellations in August is slightly larger than the cancellation amounts in September
- Tier #1 has the lowest average time duration between tier aquisition and cancellation
- Tier #2 and Tier #3 have the same average amount of days between cancellation and earning the tier

# 6. Conclusion

### 6.1 Key Takeaways

- Tier #1 has the most cancellations
- August had more cancellations than September
- Lowest amount of cancellations were in Tier #2, possibly due to customers not wanting to cancel in hope of tier promotion
- Tier #4 has the second highest number of cancellations, possibly because customer's understand that there is no higher tier
- Tier #1 has the fastest average rate of cancellations, possibly suggesting that people are quickest to cancel Tier #1 in comparison to the other tiers

### 6.2 Next Steps

Further related analysis can be conducted upon retrieving more data. Some examples of potential additional data exploration include:

1. Yearly Analysis
   - Determine annual trends of customer tier history and determine if there are certain seasonal patterns to note
2. Tier History Analysis
   - Analyze how customer's get promoted/demoted and various factors that contribute to customer tier status
3. Geographic Analysis
   - Determine if a customer's location has an effect on their retention rate

### Thank you!

Thank you for your time to understand my data analysis and process. Please feel free to reach out with any questions or concerns! 
   


