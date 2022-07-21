# Salt Lake City Real Estate

We will be analyzing SLC real estate data. The dataset contains multiple listing service (MLS) real estate transactions for houses sold in 2015-16 in zip code 84103 ([SLC avenues neighborhood](https://www.google.com/maps/place/Salt+Lake+City,+UT+84103/@40.8030372,-111.8957957,12z/data=!3m1!4b1!4m5!3m4!1s0x87525f672006dded:0x311e638d9a1a2de5!8m2!3d40.810506!4d-111.8449346)). We are primarily interested in regressing the `SoldPrice` on the house attributes (`property size`, `house size`, `number of bedrooms`, etc...). 

## The Data

1. There are 206 different variables associated with each of the 348 houses in this dataset. There are some information about real estate data on [this](http://www.utahrealestate.com/) website to get a better sense of the meanings of the column headers in the dataset.  

+ We are only keeping houses with List Price between 200,000 and 1,000,000 dollars. This is an arbitrary choice and we realize that some people are high rollers, but for our purposes we'll consider the others as outliers. 

+ We will be keeping these variables: 
`['Acres', 'Deck', 'GaragCap', 'Latitude', 'Longitude', 'LstPrice', 'Patio', 'PkgSpacs', 'PropType', 'SoldPrice', 'Taxes', 'TotBed', 'TotBth', 'TotSqf', 'YearBlt']` 

+ We will check the datatypes and convert any numbers that were read as strings to numerical values. In particular, we converted 'TotSqf' to an integer and add a column titled `Prop_Type_num` that is 

![Equation1](https://user-images.githubusercontent.com/29410712/180324387-574ecc63-5a19-4cec-805a-2d57ebd2901e.png)

+ We removed the listings with erroneous `Longitude` (one has Longitude = 0) and `Taxes` values (two have unreasonably large values).

## Exploratory Data Analysis 

We will be exploring the data set by

+ Creating a bar chart showing the breakdown of the different types of houses (single family, townhouse, condo).

![House-Type-Bar-Chart](https://user-images.githubusercontent.com/29410712/180318914-c14b7660-f64b-4ce3-8f8a-59cee094849b.png)

+ Computing the correlation matrix and use a heat map to visualize the correlation coefficients. 

![Real-Estate-Correlation-Matrix](https://user-images.githubusercontent.com/29410712/180318962-d53a7e1e-a192-420d-83f5-c25ff4590301.png)

+ Making a scatter plot matrix to visualize the correlations to determine which columns have strong correlations. 

![real-estate-scatter](https://user-images.githubusercontent.com/29410712/180319003-85635773-b851-4135-a30c-a6680a8a892f.png)

In our exploratory analysis, there are 260 observations and 18 variables in the train dataset. Based on the correlation scatterplot, the columns with the strongest correlation is the LstPrice and the SoldPrice. The plot is almost a linear line. Additionally, there are more that own a Single Family then any other Property Type.

## Analysis Tools

### Geospatial plot
Geospatial data defines specific geographical locations, either in the form of latitude and longitude coordinates or text fields with names of geographical areas, such as countries or states. Geospatial charts combine geospatial data with other forms of data to create map-based charts.

Two of the variables are the latitude and longitude of each listing. Salt Lake City is on a east-west, north-south grid, so even a simple plot of latitutde and longitude makes sense. Thus, we will be creating a scatterplot of these two variables.

![real-estate-geospatial](https://user-images.githubusercontent.com/29410712/180321451-3765f0f8-3f4f-4da2-9101-15a80d963031.png)

As we can see from the chart, the bigger the longitude and lattitude, the higher the price is for the house. Houses closer to (-111.91, 40.770) are cheaper in price.

### Linear Regression

We will now create a linear regression model to regress the Sold price on some of the other variables.

<p align="center">
  Sold Price = $β_0 + β_1x$
</p>

where $x$ is one of the other variables. 

After we run the regression, we will make a scatterplot of list price vs sold price and overlay the prediction with the regression model. 

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:              SoldPrice   R-squared:                       0.990
Model:                            OLS   Adj. R-squared:                  0.990
Method:                 Least Squares   F-statistic:                 2.611e+04
Date:                Thu, 21 Jul 2022   Prob (F-statistic):          2.97e-261
Time:                        13:50:46   Log-Likelihood:                -2927.2
No. Observations:                 260   AIC:                             5858.
Df Residuals:                     258   BIC:                             5866.
Df Model:                           1                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept   1.317e+04   2948.953      4.465      0.000    7361.378     1.9e+04
LstPrice       0.9441      0.006    161.593      0.000       0.933       0.956
==============================================================================
Omnibus:                      123.259   Durbin-Watson:                   2.015
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              881.446
Skew:                          -1.750   Prob(JB):                    3.95e-192
Kurtosis:                      11.313   Cond. No.                     1.27e+06
==============================================================================

Warnings:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 1.27e+06. This might indicate that there are
strong multicollinearity or other numerical problems.
```
![lstprice-soldprice-scatter](https://user-images.githubusercontent.com/29410712/180321828-91fe8634-1248-40b3-bfc7-f84d38117121.png)

Based on the OLS Regression, the R-squared is 0.988. This is saying that the 98.8% of the variation is explained in the model. The Beta one of LstPrice is 0.9019. In other words, as LstPrice goes up by 1, SoldPrice increases by 0.9019 points.

### Multilinear Regression

We now develop a multilinear regression model for house prices in this neighborhood. We could use this to come up with a list price for houses coming on the market, so will not be including the list price in our model and also ignore the categorical variable Prop_Type. Our model is now in the form:

<p align="center">
  Sold Price = $β_0 + β_1x_1 + β_2x_2 + \cdots + β_nx_n$
</p>

where $x_i$ are predictive variables.

Next we look at the difference between list price and sold price explicitly and calculate two new columns for our dataset. `DiffPriceAbsolute` and `DiffPriceRelative`.

* `DiffPriceAbsolute` - This is difference between sold price and list price. If it is positive, that means the house sold for more than it was listed at.
* `DiffPriceRelative` - This is the relative difference between sold price and list price. A value of 1.1 here means that the house sold for 110% of the asking price, and 0.9 means the house sold for 90% of the asking price.

After that, we create two new models. One to predict `DiffPriceAbsolute`, and one to predict `DiffPriceRelative` and use the same predictive variables as i the last model. From there, we compare which of the two new models makes better predictions.

From the two models, we will train two models to predict `DiffPriceAbsolute` and `DiffPriceRelative` based on just `SoldPrice` and create a scatterplot for each model.

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:              SoldPrice   R-squared:                       0.838
Model:                            OLS   Adj. R-squared:                  0.835
Method:                 Least Squares   F-statistic:                     218.7
Date:                Thu, 21 Jul 2022   Prob (F-statistic):           4.45e-97
Time:                        13:51:02   Log-Likelihood:                -3291.9
No. Observations:                 260   AIC:                             6598.
Df Residuals:                     253   BIC:                             6623.
Df Model:                           6                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept   2.765e+05   2.94e+05      0.942      0.347   -3.02e+05    8.55e+05
Acres       1.169e+05   6.15e+04      1.900      0.059   -4276.513    2.38e+05
Taxes         58.6029      5.466     10.721      0.000      47.838      69.368
TotBed      1.286e+04   6045.229      2.127      0.034     951.492    2.48e+04
TotBth      6050.4777   8140.361      0.743      0.458   -9981.025    2.21e+04
TotSqf        45.5661      9.878      4.613      0.000      26.113      65.019
YearBlt      -96.9146    151.794     -0.638      0.524    -395.855     202.025
==============================================================================
Omnibus:                       30.768   Durbin-Watson:                   2.174
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               43.356
Skew:                           0.759   Prob(JB):                     3.85e-10
Kurtosis:                       4.302   Cond. No.                     2.90e+05
==============================================================================

Warnings:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 2.9e+05. This might indicate that there are
strong multicollinearity or other numerical problems.
```
```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:      DiffPriceAbsolute   R-squared:                       0.261
Model:                            OLS   Adj. R-squared:                  0.244
Method:                 Least Squares   F-statistic:                     14.92
Date:                Thu, 21 Jul 2022   Prob (F-statistic):           1.32e-14
Time:                        13:51:02   Log-Likelihood:                -2927.3
No. Observations:                 260   AIC:                             5869.
Df Residuals:                     253   BIC:                             5894.
Df Model:                           6                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept   2.867e+04   7.22e+04      0.397      0.692   -1.14e+05    1.71e+05
Acres      -1.235e+04   1.51e+04     -0.815      0.416   -4.22e+04    1.75e+04
Taxes         -1.1223      1.345     -0.834      0.405      -3.771       1.527
TotBed       528.3073   1487.544      0.355      0.723   -2401.240    3457.854
TotBth     -2969.8850   2003.091     -1.483      0.139   -6914.743     974.973
TotSqf        -4.9697      2.431     -2.045      0.042      -9.756      -0.183
YearBlt       -9.4079     37.352     -0.252      0.801     -82.968      64.152
==============================================================================
Omnibus:                       98.703   Durbin-Watson:                   2.026
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              618.218
Skew:                          -1.371   Prob(JB):                    5.70e-135
Kurtosis:                      10.039   Cond. No.                     2.90e+05
==============================================================================

Warnings:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 2.9e+05. This might indicate that there are
strong multicollinearity or other numerical problems.
```
```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:      DiffPriceRelative   R-squared:                       0.079
Model:                            OLS   Adj. R-squared:                  0.057
Method:                 Least Squares   F-statistic:                     3.618
Date:                Thu, 21 Jul 2022   Prob (F-statistic):            0.00184
Time:                        13:51:02   Log-Likelihood:                 486.61
No. Observations:                 260   AIC:                            -959.2
Df Residuals:                     253   BIC:                            -934.3
Df Model:                           6                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
Intercept      1.0221      0.143      7.131      0.000       0.740       1.304
Acres         -0.0145      0.030     -0.483      0.629      -0.074       0.045
Taxes       9.371e-07   2.67e-06      0.351      0.726   -4.32e-06    6.19e-06
TotBed         0.0025      0.003      0.838      0.403      -0.003       0.008
TotBth        -0.0080      0.004     -2.022      0.044      -0.016      -0.000
TotSqf     -3.941e-06   4.82e-06     -0.817      0.415   -1.34e-05    5.56e-06
YearBlt    -1.261e-05   7.41e-05     -0.170      0.865      -0.000       0.000
==============================================================================
Omnibus:                      119.899   Durbin-Watson:                   2.053
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              840.861
Skew:                          -1.697   Prob(JB):                    2.57e-183
Kurtosis:                      11.130   Cond. No.                     2.90e+05
==============================================================================

Warnings:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 2.9e+05. This might indicate that there are
strong multicollinearity or other numerical problems.
```
![DiffPriceAbsolute](https://user-images.githubusercontent.com/29410712/180322332-5916c8b9-ce6c-42ff-ac7e-08d0a55a058e.png)

![DiffPriceRelative](https://user-images.githubusercontent.com/29410712/180322345-9225101e-d08e-45b7-a23d-96319290064c.png)

The multilinear regression model has a lower R squared than the simple linear regression model including the List Price. Therefore, there is more of the variation explained by the simple regression model. Additionally, based on the two models, the model that predicts the DiffPriceAbsolute is better than the model that predicts the DiffPriceRelative because the R squared is greater. Furthermore, from the different trained plots, the plot for the DiffPriceAbsolute has more plots closer to the linear line, thus shows the why more of the variation is explained. Although DiffPriceAbsolute is better, it still doesn't have a high R squared.

#### Incorporating a Categorical Variable

Above, we considered houses, townhouses, and condos together, but here 

We will now distinguish the difference between houses, townhouses, and condows. Consider the two regression models: 

<p align="center">
  Sold Price = $β_0 + β_1$ (Prop_Type_num)
</p>

<p align="center">
  Sold Price = $β_0 + β_1$ (Prop_Type_num) $+ β_2$ (TotSqf)
</p>

From the first model, it would appear that Property type is significant in predicting the sold price. On the other hand, the second model indicates that when we take into account total square footage, property type is no longer predictive. 

We will also make a scatterplot of TotSqf vs. SoldPrice where the house types are colored differently to illustrate your explanation. 

![TotSqf-SoldPrice-Scatter](https://user-images.githubusercontent.com/29410712/180322473-26098837-49d0-4681-899f-77414775db27.png)

Based on the scatterplot, there seems to be a strong positive correlation between TotSqf and SoldPrice. The greater the total square feet, the higher the price. It is the same for all types of properties.
