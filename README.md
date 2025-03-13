# Snowflake-Data-Cleaning
This project involved several techniques to clean messy data in `Snowflake`.

## Project Scenario
For a marketing campaign, we need to find a list of inactive customers (didn't make any transactions in the last 90 days).
Available customers data has several challenges like duplicated customers, missing emails, merged columns, non-standarized phone numbers and wrong data types.
In addition, additional fields need to be calculated.

The task is to reformat and clean the data using SQL functions in `Snowflake` before finding the target list of inactive customers.

### `Task1:` Data Loading

There are several ways to load data into a `Snowflake` table, in this case we used SQL Insert Statements for this project.

![image](https://github.com/user-attachments/assets/26e443b8-f857-454f-a903-a31ea2ea043e)

The data contains the following column fields:

- **`Name:`** Customer's full name 
- **`Phone:`** Customer's phone number
- **`Email:`** Customer's e-mail address
- **`Address:`** Customer's physical address
- **`PostalZip:`** Postal code associated with the address
- **`Region:`** Region, state or province where the customer resides
- **`Country:`** Country of residence of the customer  
- **`Company:`** Name of the company the customer is associated with
- **`LastTransaction:`** Timestamp of customer's last recorded transaction.
- **`DOB:`** Customer's Date of Birth

### `Task2:` Investigate Data Quality Issues

Checking the data we found missing customer names, emails and phone numbers: 

![image](https://github.com/user-attachments/assets/f2f30aee-1986-468c-b6ac-b5caf300f050)

We also found duplicate email address associated with the same person:

![image](https://github.com/user-attachments/assets/cf1553d0-ab2c-4878-8bb8-32bffb3034e8)

**Name**, **Phone** and **DOB** fields need formatting:

![image](https://github.com/user-attachments/assets/239ef225-f11a-4eae-a231-019f76d77cd8)

### `Task3:` Remove Unwanted Characters

The `CONCAT()` function is used to help spot spaces:
![image](https://github.com/user-attachments/assets/9c44ecbd-42e1-4474-b421-6a513bfa91d7)

The `TRIM()` function is used to remove leading and trailing spaces and zeros.
![image](https://github.com/user-attachments/assets/cc839f85-6226-43dc-a262-9b7c3249bd3a)

### `Task4:` Extract First and Last Name from `NAME` Column

The `SPLIT_PART()` function is used to split first and last names, including previously used `TRIM()` function:
![image](https://github.com/user-attachments/assets/64cbb1e6-85d6-4de6-8b6c-2fe5c9d01fbb)

### `Task5:` Standarize `PHONE` Column

The `LTRIM()` function is used to remove leading zeros and plus signs.
![image](https://github.com/user-attachments/assets/8d3dfcee-875a-4b6d-898e-c3a6069965bc)

### `Task6:` Extract DATE from a Text Column

Our columns of interest are `DOB` (Date of Birth) and `LASTTRANSACTION`. The last one is used to filter 90days old customer transactions (Inactive Customers):

![image](https://github.com/user-attachments/assets/42974e24-9933-4552-8bbb-08a1ad819709)

We then apply `TO_DATE()` function to convert the text columns to an standarized date format:

![image](https://github.com/user-attachments/assets/2b297418-7832-4897-9de9-da1d528844ab)

### `Task7:` Add new column `DAYS_SINCE_LAST_TRANSACTION`

Our end goal for this project is to find the list of inactive customers, those who didn't make any transactions in the last 90 days.

We add a new calculated column `DAYS_SINCE_LAST_TRANSACTION` that shows the number of days of customers' last transaction.

It is calculated as a difference between `CURRENT_DATE` and `LASTTRANSACTION`:

![image](https://github.com/user-attachments/assets/8a3e1328-a79c-46eb-a8ec-32fdf5356a87)

In order to find inactive customers we add a `WHERE` clause to filter (DAYS_SINCE_LAST_TRANSACTION > 90):

![image](https://github.com/user-attachments/assets/74a690ed-fac3-48b9-97e4-097da68c4161)

### `Task8:` Calculate Customers' Age

Customer's Age is calculated as a difference between `CURRENT_DATE` and `DOB`:

![image](https://github.com/user-attachments/assets/d18517a0-af5c-4268-9dca-c266d3180c75)

### `Task9:` Handling Missing Values

Below is the final query with all applied SQL functions:

![image](https://github.com/user-attachments/assets/e995686c-72ab-453d-ae75-0d9cf01c30e9)

This is the final results of inactive customers:

![image](https://github.com/user-attachments/assets/8785a672-d622-4760-b287-97145785d3e7)

After analizing the data we see that email is the best contact point of customers, so we will keep customers with filled email.

We will filled missing company values with 'N/A' using `IFF()` function

The final query below:

![image](https://github.com/user-attachments/assets/cc60fe4b-9c82-4866-8e9f-b3e2693485d3)

The final results after handling missing values:

![image](https://github.com/user-attachments/assets/2bd25fc6-9ea9-4ab4-a9c2-b29fbc124f88)

### `Task10:` Eliminate Duplicates

In order to eliminate duplicates, we first need to detect them and select an unique column, in our case we select `EMAIL` as our unique column. We use SQL query on the Raw Data to find all duplicated emails. We don't need to worry about **NULL** values, because we already dealed with them.

![image](https://github.com/user-attachments/assets/a9030c78-917c-4f05-b2e7-0abf7da6451e)

Once duplicated emails (rows) are detected, we filter it and keep the records with the latest transaction date.

In this case we use `RANK()` function to assign a unique rank to each row, that rank is done based on email. For each duplicated email an unique rank is assigned
where rank 1 refers to the latest transaction date.

![image](https://github.com/user-attachments/assets/87c98b91-b5de-4116-b22c-e3486e921963)

To filter with rank = 1, `QUALIFY`clause must be used instead of `WHERE`.

![image](https://github.com/user-attachments/assets/e2c59168-6f16-4962-ace6-f9711ee9ba85)

### `Task11:` Create View and export Inactive Customers

```sql
WITH LISTID AS (
    SELECT 
        ID,
        RANK() OVER (PARTITION BY EMAIL ORDER BY TO_DATE(LASTTRANSACTION, 'AUTO') DESC) AS RANK
        FROM CUSTOMERS
        QUALIFY RANK = 1
)
SELECT 
    ID, 
	SPLIT_PART(TRIM(NAME,' 0'),', ', 1) AS FIRST_NAME, 
    SPLIT_PART(TRIM(NAME,' 0'),', ', 2) AS LAST_NAME, 
    EMAIL,
    TO_DATE(DOB,'MMMM DD, YYYY') AS DOB,
	TO_DATE(LastTransaction,'AUTO') AS LastTransaction,
	DATEDIFF(days, TO_DATE(LastTransaction,'AUTO'), current_date()) AS DaysSinceLastTrans,
	DATEDIFF(days, TO_DATE(DOB,'MMMM DD, YYYY'), current_date()) AS Age,
    LTRIM(PHONE,'0+') AS Standarized_Phone,
    ADDRESS, 
    REGION, 
    COUNTRY,
    IFF(COMPANY IS NULL OR COMPANY = '', 'N/A', COMPANY) AS COMPANY
FROM
    CUSTOMERS
WHERE
    DaysSinceLastTrans > 90 
    AND NOT (EMAIL IS NULL OR EMAIL = '')
    AND ID IN (SELECT ID FROM LISTID)
```







 
