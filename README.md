
# Module 2 Final Project

Student name: John Cho
Student pace: full time online
Scheduled project review date/time: 2/25/20
Instructor name: Rafael Carrasco
Blog post URL: https://medium.com/@johnnyboyee/my-foray-into-data-science-continued-98bcfba76316

## Introduction

For this project, we worked with the King County House Sales dataset, which has been modified to make it a bit more fun and challenging.  The dataset can be found in the file `"kc_house_data.csv"`, in this repo.

The description of the column names can be found in the column_names.md file in this repository. As with most real world data sets, the column names were not perfectly described, so a little research and best judgment had to be applied.

Tasks involved cleaning, exploring and modeling this dataset with a multivariate linear regression to predict the sale price of houses as accurately as possible.

## Final Project Summary

There are 2 notebook files:
1. student_Clean_1of2.ipynb - Data Cleaning
2. student_EDA_2of2.ipynb - Exploratory Data Analysis and Regression Model

## Data Cleaning

### Invalid / null values, non-useful columns

- Initial dataset included 21,597 rows (properties) and 21 columns (target/dependent variable and features/independent variables).
- Null values: 'waterfront' (2376), 'view' (63), 'yr_renovated' (3842).
- Dropped 'id' column by making it the index.
- Confirmed 177 'id' duplicates were subsequent sales of the same property: 176 properties were sold twice and 1 sold 3 times.
- Dropped 'waterfront' column along with 146 waterfront properties.
- Dropped 'view' column due to: description did not make sense with populated values, over 90% of dataset had a value of zero/null, distribution was very non-normal and weak price correlation.
- Dropped 'yr_renovated' column due to almost 97% containing zero/null values.
- Discovered 'sqft_basement' had 450 '?' values which were filled in with the difference between 'sqft_living' and 'sqft_above'.. confirmed entire dataset had no discrepancies with sqft_basement + sqft_above = sqft_living.

### Dropping outliers

- Price outliers > mean+3std ($1.56M): 404
- Sqft_lot > 650k: 9
- Sqft_lot15 > 400k: 6
- Bedrooms > 10: 2
- Bathrooms > 7: 1
- Sqft_living > 7000: 4

## EDA

### Question 1: Are the TIME related features helpful in predicting our target (price) in this project?
- Our 3 time related features (date sold, year built, year renovated) may not be as helpful as they seem at first glance. Since we did not have prior sales data, we could not compare sales price differences from one point in time to another. In addition, ~3% of the properties were renovated but we do not know which features might have changed.
- Our range of selling dates is very short, ~1 year: 5/2/14 - 5/27/15
- Dropped the 'date' and 'yr_built' columns, replacing them with a new 'age' column (sold 'date' - 'yr_built').
- **Note: fun figuring out method to get 'year fractions' since the sell date column had exact day/month/year - my method was not perfectly precise as when adding the 'fractional year day' I simply used day/365 instead of day/(exact # of days in month).
- **Easter egg: 11 properties mistakenly claimed to be built in 2015 yet sold in 2014.**
- Curiosity check: 'newer' (built or renovated since 2000) homes had an average higher price of almost $125k.

### Question 2: Should we be concerned about multiple sales for the same property in such a short timeframe?
- Short answer: yes! Out of the 176 properties sold more than once, the average price difference (profit) was $135k.
- These houseflippers have created 'bad' data since these properties have identical features and significantly different selling prices.
- **Easter egg: 2014 was the mode for 'yr_renovated', or the most popular renovation year.**
- All 'flipped' properties were combined into a single row with its mean for 'price' and 'age'. (177 rows dropped)

### Question 3: Should we be concerned about urban/rural properties and their locations?
- Out of the 3 location variables (latitude, longitude, zipcode) we chose to only use latitude.
- Since many of the sqft (living, above, basement, lot, lot15, living15) exhibit strong multicollinearity, dropping all but 1 or 2 would be necessary. See if we can get other insights that can be used instead.
- Basements: since 61% of homes have no basement (value=0) and the value is contained in sqft_living, we chose to drop the column. Homes with a basement have a higher selling price by ~$100k more on average; created a basement yes/no (1/0) column instead.
- Urban vs rural: determined optimal threshold ratio of 0.32 (living / lot) or roughly 1:2 (sqft_living:sqft_lot) where average price difference was the greatest between the 2 groups. Based on assumption that an equivalent featured 'urban' home would sell for significantly less than its 'rural' brother.
- Dropped 'sqft_lot/15' columns and created urban (1/0) column where >=0.32 ratio was considered 'urban' and <0.32 considered 'rural'. Observed average price difference of $135 between both groups.
- **Note: The significantly higher averages for 'rural' lot square footage became very apparent once these groups were separated. Mean and std were 3-4x and 15-20x greater, respectively. Highest sqft_lot value was 40x greater (even after feature outliers dropped).**

### Regression Model: Feature Selection
- Round 1: dropped condition, age, sqft_above.
- Round 2: log transformed sqft_living/15, bedrooms, grade and used one hot encoding for floors. Performed min-max scaling on lat, mean normalization on sqft_living/15, grade, bathrooms. Dropped sqft_living15 and bathrooms due to multicollinearity.
- Round 3: dropped bedrooms, grade.
- Final model: sqft_living, lat, floors (bin), urban, basement.

### Model Validation
- Using train-test split, it appears the standard 60/40 was appropriate to achieve the lowest mean squared errors.
- Average price prediction error of $170k with 89.5% attributable to our model.
- Pretty good Skew (0.716), Kurtosis (4.804) and Condition No. (8.81) 
- Q-Q plot shows decent normality (long right tail) and scatterplot of residuals demonstrate homoscedasticity (maybe?)
- Updated model by dropping more price outliers (983 properties > $1M) and model improved dramatically
- Q-Q plot looks almost perfect and residuals appear homoscedastic with a negative slope, OLS regression results indicate R2 = 91.6%, improved Skew (0.102), Kurtosis (3.340) and Condition No. (8.89).


## Summary

In closing, what we have learned over the course of this project demonstrates the importance of:
* Linear relationships between feature and target variables
* Distribution normality of all variables
* Having lots of datapoints representing low, mid, high values; convert to a binary feature if lacking
* How regression models deal poorly with features representing continuous data containing too many zero values
* How data scientists can 'manipulate' data to achieve more desirable results

## Future Work
* Create 3 separate regression models for urban, rural, waterfront. Separating properties into groups with more similar feature patterns should yield more accurate price predictions.
* Possibly define a 4th category for properties with HUGE lots. These will often be farms or possibly large land sales where the home features would have even less of an effect on final price. (ie. 2k sqft home with a 1M+ sqft lot?)
* Dive deeper into geographical features. Create a column indicative of how strong its location is to demand, which has the greatest effect on housing price, by far.