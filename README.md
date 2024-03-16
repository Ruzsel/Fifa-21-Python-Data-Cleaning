# Fifa 21 Python Data Cleaning
## Overview
This repository contains the code to clean up the FIFA 21 dataset game. The purpose of this project is to clean and standardize the value format of several columns that originally varied. FIFA21 dataset contains 18K+ records of player data and 77 Columns. Each record represents a unique player and includes various attributes
## Issues found in the data
- Inconsistent value formats : Several columns have inconsistent value formats, which could potentially disrupt operations such as grouping by, filtering, or aggregating.
- Missing Values : Some columns have missing values that need to be handled differently.

## LinkedIn
[**Fairuz Mujahid Annabil**](https://www.linkedin.com/in/fairuzmujahidannabil/)

## Tools Used
1. Python : used Numpy and Pandas Library for Cleaning
2. Visual Code Studio : IDE for Data Exploration and Cleaning
## Columns Explanation
For descriptions of all columns, you can refer to a file named "FIFA21 Column Explanation"
## Data Analysis
Data Analysis Table of Contents
- [Importing Libraries](#importing-libraries-and-read-the-csv-file)
- [Cleanning Club Column](#1-cleanning-club-column)
- [Cleanning Contract Column](#2-cleanning-contract-column)
- [Cleanning Height Column](#3-cleanning-height-column)
- [Cleanning Weight Column](#4-cleanning-weight-column)
- [Cleanning Loan Date End Column](#5-cleanning-loan-date-end-column)
- [Cleanning W/F Column](#6-cleanning-wf-column)
- [Cleanning SM Column](#7-cleanning-sm-column)
- [Cleanning Hits Column](#8-cleanning-hits-column)
- [Cleaning Value, Wage, Release Clause Column](#9-cleaning-value-wage-release-clause-column)
### Importing Libraries and Read the csv file
```python
import numpy as np
import pandas as pd
```
```python
df = pd.read_csv('fifa21 data.csv')
pd.set_option('display.max_columns', None)
df
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/cb640c31-f0ad-49eb-8ebc-155cbedc7ab1)
```python
df.info()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/0e059a5a-9a8e-468f-a3ce-2d91bb812f08)

### Data Cleanning
create a copy of dataframe
```python
data = df.copy()
data.head(5)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/e86be81b-dd4a-430c-adda-f5550d023a93)

### 1. Cleanning Club Column
```python
data['Club'].dtype
```
```python
data['Club'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/d3f7f720-4c30-42e3-9d00-9b358b72a740)
There are many whitespace characters in some club names in the Club column. Since the whitespace characters are both at the beginning and end of the string, using .strip() would be appropriate to remove them.
```python
data['Club'] = data['Club'].str.strip()
data['Club'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/1c3de9bf-ee72-413e-960d-65c1f540498e)

### 2. Cleanning Contract Column
```python
data['Contract'].dtype
```
```python
data['Contract'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/dde86699-e538-4e38-bf27-fd44f0586715)
using `.loc` to search for values in the 'Contract' column for each row where the value contains the string 'On Loan' or 'Free'.
```python
for index, kontrak in data.loc[(data['Contract'].str.contains('On Loan')) | (data['Contract'].str.contains('Free')), 'Contract'].items():
    print(kontrak)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/cc5971a3-03f3-49ba-97d5-b1f55e8da431)
Declare a function `extract_contract` that accepts one parameter `contract`, which is used to create 3 new columns: 'Contract Start', 'Contract End', and 'Contract Length (years)' with values taken from the 'Contract' column.
```python
def extract_contract(contract):
    if contract == 'Free' or 'On Loan' in contract:
        start_date = np.nan
        end_date = np.nan
        contract_length = 0
    else:
        start_date, end_date = contract.split(' ~ ')
        start_year = int(start_date[:4])
        end_year = int(end_date[:4])
        contract_length = end_year - start_year
    return start_date, end_date, contract_length

new_columns = ['Contract Start', 'Contract End', 'Contract Length(years)']
new_data = data['Contract'].apply(lambda x: pd.Series(extract_contract(x)))

for i in range(len(new_columns)):
    data.insert(loc=data.columns.get_loc('Contract')+1+i, column=new_columns[i], value=new_data[i])
```
```python
data[['Contract', 'Contract Start', 'Contract End', 'Contract Length(years)']].sample(10)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/ca655152-9c82-45da-be56-4e23e9d5bda9)

Declare a function `Categorize_Contract` that categorizes values in the 'Contract' column into three categories: 'Free', 'On Loan', and 'Contract'. Then create a new column 'Contract Status' to the DataFrame data to the right of the 'Contract Length (years)' column. The values for this new column are generated by applying the `Categorize_Contract` function to each value in the 'Contract' column.
```python
def Categorize_Contract(contract):
    if contract == 'Free':
        return 'Free'
    elif 'On Loan'in contract:
        return 'On Loan'
    else:
        return 'Contract'

data.insert(data.columns.get_loc('Contract Length(years)')+1, 'Contract Status', data['Contract'].apply(Categorize_Contract))
data.sample(4)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/008d1f5a-25f9-494e-965f-174d03c9c7db)

```python
data[['Contract', 'Contract Start', 'Contract End', 'Contract Length(years)', 'Contract Status']].sample(5)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/ebb4c385-a3ce-41d7-b295-d2bb92d089dd)

### 3. Cleanning Height Column
```python
data['Height'].dtype
```
```python
data['Height'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/044eb51c-857b-4606-8c24-c13120dbd9bd)
Declare a function `transform_height` that accepts one parameter used to remove the string 'cm' and convert the height value from inches to centimeters, resulting integer data type only
```python
def transform_height(x):
    if 'cm' in x:
        return int(x.strip('cm'))
    else:
        feet, inches = x.split("'")
        total_inches = int(feet)*12 + int(inches.strip('"'))
        return round(total_inches * 2.54)
    
data['Height'] = data['Height'].apply(transform_height)
data['Height'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/6bba56b6-a030-4448-b4de-d325c998880e)

rename Height Column to Height(Cm)
```python
data = data.rename(columns={'Height':'Height(Cm)'})
data.sample(1)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/e63ccdb0-e107-4819-9bfd-e1db4a715c69)

### 4. Cleanning Weight Column
```python
data['Weight'].dtype
```
```python
data['Weight'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/f0a1711c-b227-429e-8f96-71179aeb12ef)

Declare a function `transform_weight` that removes the string 'kg' and 'lbs' characters, then converts the weight from pounds to kilograms, resulting integer data type
```python
def transform_weight(x):
    if 'kg' in x:
        return int(x.strip('kg'))
    else:
        pounds = int(x.strip('lbs'))
        return round(pounds/2.205)
    
data['Weight'] = data['Weight'].apply(transform_weight)
data['Weight'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/05892e11-3aca-4c46-94d8-037996215028)

rename 'weight' column to 'weight(kg)'
```python
data = data.rename(columns={'Weight':'Weight(kg)'})
data.sample(7)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/ad7a44a4-331d-4524-87f4-8dbc33cafc68)

### 5. Cleanning Loan Date End Column
```python
data['Loan Date End'].dtype
```
```python
data['Loan Date End'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/221048ab-6eb0-467e-a09d-e17ba506eaa8)

```python
player_onloan = data[data['Contract Status'] == 'On Loan']
player_onloan[['Contract','Contract Status', 'Contract End', 'Loan Date End']]
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/23ceefed-5dd0-47c8-b670-1b267f12d855)

There are many missing values in the `Loan Date End` column because not all players in this dataset are players in the "On Loan" status. There are 1013 rows of data from players in the "On Loan" status, and I think this column might be useful in the future, so I won't delete it.

### 6. Cleanning W/F Column
```python
data['W/F'].dtype
```
```python
data['W/F'].unique()
```
```python
data['W/F'] = data['W/F'].str.replace('★', '')
data['W/F'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/45b87b3e-4bf5-4a5c-bc03-a2f9c7e3e966)

### 7. Cleanning SM Column
```python
data['SM'].dtype
```
```python
data['SM'].unique()
```
```python
data['SM'] = data['SM'].str.replace('★', '')
data['SM'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/7f2a3785-4615-40ca-a476-71dae851d036)

### 8. Cleanning Hits Column
```pyton
data['Hits'].dtype
```
```python
data['Hits'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/569171a4-438d-4533-b950-6f8d816a5283)

Declare a function `convert_hits` that replaces null values with 0, removes the string 'k' character, multiplies it by 1000, and converts it to an integer data type. Finally, if there's a float or integer value, it will be converted to an integer value only.
```python
def convert_hits(x):
    if pd.isnull(x):
        return 0
    elif 'K' in str(x):
        number_only = str(x).replace('K', '')
        number_float = float(number_only)
        number_int = int(number_float * 1000)
        return number_int
    elif isinstance(x, float) or isinstance(x, int):
        return int(x)
    else:
        return int(float(x))

    
data['Hits'] = data['Hits'].apply(convert_hits)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/793cf092-3036-4bf4-baf9-1d18a68ef0a2)
```python
data['Hits'].dtype
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/59b62d5d-6165-4123-b761-7bcd7d0481de)
```python
data['Hits'].unique()
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/9f6a9eb2-f701-4f03-8b19-aac30addd220)
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/43c2b2e8-6b69-4560-bb23-8ebc0d24e9f0)

### 9. Cleaning Value, Wage, Release Clause Column
Since I want to use the 'Value', 'Wage', 'Release Clause' columns as aggregate columns in Power BI, it would be easier if I convert the values of these three columns into integers only value and add the euro denomination in the column names.
```python
def convert_euro_to_number(value):
    if '€' in value:
        if 'M' in value:
            return int(float(value.replace('€', '').replace('M', '')) * 1e6)
        elif 'K' in value:
            return int(float(value.replace('€', '').replace('K', '')) * 1e3)
        else:
            return 0
    return int(value)
```
The function convert_euro_to_number is designed to convert Euro-formatted values into numeric format. It checks for the presence of the Euro symbol ('€') in the value and then handles cases where the value is in millions ('M') or thousands ('K'). Ultimately, it simplifies the conversion process, making it easier to analyze Euro values numerically.

Applying `convert_euro_to_number` function to 'Value', 'Wage', and 'Release Clause' Column
```python
data['Value'] = data['Value'].apply(convert_euro_to_number)
data['Wage'] = data['Wage'].apply(convert_euro_to_number)
data['Release Clause'] = data['Release Clause'].apply(convert_euro_to_number)
```

Renaming 'Value', 'Wage', and 'Release Clause' Column
```python
data.rename(columns={'Value':'Value(Euro)'}, inplace=True)
data.rename(columns={'Wage':'Wage(Euro)'}, inplace=True)
data.rename(columns={'Release Clause':'Release Clause(Euro)'}, inplace=True)
```
```python
data.sample(5)
```
![image](https://github.com/Ruzsel/Fifa-21-Python-Data-Cleaning/assets/150054552/d98b7ef8-490a-4270-89a6-f371eaf76be6)

## Finally Save new data into CSV file.
```python
data.to_csv('Fifa21_fix.csv', index=False)
```











