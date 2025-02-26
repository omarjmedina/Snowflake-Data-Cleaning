# Snowflake-Data-Cleaning
This project involed several techniques to clean messy data in `Snowflake`.

## Project Scenario
For a marketing campaign, I have been assigned to find the list of inactive customers (didn't make any transactions in the last 90 days).
Available customers data has several challenges like duplicated customers, missing emails, merged columns, non-standarized phone numbers and wrong data types.
In addition, additional fields need to be calculated.

The task is to reformat and clean the data using SQL functions in `Snowflake` before finding the target list of inactive customers.

### `Task1:` Data Loading

There several ways to load data into a `Snowflake` table, in this case we used SQL Insert Statements for this project.

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
