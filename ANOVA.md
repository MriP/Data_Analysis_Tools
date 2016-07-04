
# Determine if a country's female employment rate is affected by the polity score of that country.

## Data

Data for this study comes from the Gapminder World Dataset collected by the Gapminder Foundation. The Gapminder World Dataset contains data collected from more than 200 countries/areas for more 500 variables.


## Description of Variables

Below is the description of the variables

1. **Democracy Score** (variable code: polityscore, Unit: Scale -10 being lowest, 10 highest) - Score from -10 to 10.
2. **Female Employment Rate** (variable code: femaleemployrate, Unit: Percentage) - Employed females (age > 15) as a percentage of the total female population.

**Female Employment Rate** is the response variable and **Democracy Score** is the primary explanatory variables.

### Import the libraries and loading the data into a pandas dataframe


    import numpy as np
    import pandas as pd
    import statsmodels.api as sm
    import statsmodels.formula.api as smf
    import seaborn as seaborn
    import pylab
    
    # bug fix for display formats to avoid run time errors
    pd.set_option('display.float_format', lambda x:'%.3f'%x)
    
    # Loading the data 
    data = pd.read_csv('gapminder.csv')
    
    

### Converting variables to numeric


    # convert variables to numeric format using convert_objects function
    data['polityscore'] = pd.to_numeric(data['polityscore'], errors='coerce')
    data['femaleemployrate'] = pd.to_numeric(data['femaleemployrate'], errors='coerce')
    

### First few observations of the dataset


    data.head()




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>incomeperperson</th>
      <th>alcconsumption</th>
      <th>armedforcesrate</th>
      <th>breastcancerper100th</th>
      <th>co2emissions</th>
      <th>femaleemployrate</th>
      <th>hivrate</th>
      <th>internetuserate</th>
      <th>lifeexpectancy</th>
      <th>oilperperson</th>
      <th>polityscore</th>
      <th>relectricperperson</th>
      <th>suicideper100th</th>
      <th>employrate</th>
      <th>urbanrate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td></td>
      <td>.03</td>
      <td>.5696534</td>
      <td>26.8</td>
      <td>75944000</td>
      <td>25.600</td>
      <td></td>
      <td>3.65412162280064</td>
      <td>48.673</td>
      <td></td>
      <td>0.000</td>
      <td></td>
      <td>6.68438529968262</td>
      <td>55.7000007629394</td>
      <td>24.04</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>1914.99655094922</td>
      <td>7.29</td>
      <td>1.0247361</td>
      <td>57.4</td>
      <td>223747333.333333</td>
      <td>42.100</td>
      <td></td>
      <td>44.9899469578783</td>
      <td>76.918</td>
      <td></td>
      <td>9.000</td>
      <td>636.341383366604</td>
      <td>7.69932985305786</td>
      <td>51.4000015258789</td>
      <td>46.72</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>2231.99333515006</td>
      <td>.69</td>
      <td>2.306817</td>
      <td>23.5</td>
      <td>2932108666.66667</td>
      <td>31.700</td>
      <td>.1</td>
      <td>12.5000733055148</td>
      <td>73.131</td>
      <td>.42009452521537</td>
      <td>2.000</td>
      <td>590.509814347428</td>
      <td>4.8487696647644</td>
      <td>50.5</td>
      <td>65.22</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Andorra</td>
      <td>21943.3398976022</td>
      <td>10.17</td>
      <td></td>
      <td></td>
      <td></td>
      <td>nan</td>
      <td></td>
      <td>81</td>
      <td></td>
      <td></td>
      <td>nan</td>
      <td></td>
      <td>5.36217880249023</td>
      <td></td>
      <td>88.92</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Angola</td>
      <td>1381.00426770244</td>
      <td>5.57</td>
      <td>1.4613288</td>
      <td>23.1</td>
      <td>248358000</td>
      <td>69.400</td>
      <td>2</td>
      <td>9.99995388324075</td>
      <td>51.093</td>
      <td></td>
      <td>-2.000</td>
      <td>172.999227388199</td>
      <td>14.5546770095825</td>
      <td>75.6999969482422</td>
      <td>56.7</td>
    </tr>
  </tbody>
</table>
</div>



## Binning the Explanatory variable

Since we are doing an ANOVA analysis, we would need to bin (categorize) the explantory variable. Here the explantory variable **Democracy Score** has values from -10 to 10. We would categorize it into four categories.

Democracy Score between -10 and -6 (both inclusive) : Categorized as 0 (_Authoritarian Regimes_)

Democracy Score between  -5 and -1 (both inclusive) : Categorized as 1 (_Hybrid Regimes_)

Democracy Score between   0 and  5 (both inclusive) : Categorized as 2 (_Flawed Democracies_)

Democracy Score between   6 and 10 (both inclusive) : Categorized as 3 (_Full Democracies_)

Ideas for this classification have been taken from the Democracy Index entry in Wikipedia available [here](https://en.wikipedia.org/wiki/Democracy_Index).


    def categorize_dem_score(x):
        if x['polityscore'] <= -10 and x['polityscore'] <= -6:
            return 0
        elif x['polityscore'] <= -5 and x['polityscore'] <= -1:
            return 1
        elif x['polityscore'] <= 0 and x['polityscore'] <= 5:
            return 2
        elif x['polityscore'] <= 6 and x['polityscore'] <= 10:
            return 3
        
    data['bin_polityscore'] = data.apply(lambda x: categorize_dem_score(x), axis =1)
    data_complete = data[['bin_polityscore','femaleemployrate']].dropna()

### View of the data now


    data_complete.head()




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bin_polityscore</th>
      <th>femaleemployrate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.000</td>
      <td>25.600</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3.000</td>
      <td>31.700</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.000</td>
      <td>69.400</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3.000</td>
      <td>34.200</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1.000</td>
      <td>56.200</td>
    </tr>
  </tbody>
</table>
</div>



### ANOVA

### Hypothesis Testing

**Null Hypothesis**: There is no association between the female employment rate and the polity score of the country.

**Alternative Hypothesis**: The female employment rate is affected by the polity score of the country.



    model = smf.ols(formula='femaleemployrate ~ C(bin_polityscore)', data=data_complete)
    results = model.fit()
    print(results.summary())
    

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:       femaleemployrate   R-squared:                       0.041
    Model:                            OLS   Adj. R-squared:                  0.003
    Method:                 Least Squares   F-statistic:                     1.075
    Date:                Mon, 04 Jul 2016   Prob (F-statistic):              0.365
    Time:                        09:52:20   Log-Likelihood:                -345.07
    No. Observations:                  80   AIC:                             698.1
    Df Residuals:                      76   BIC:                             707.7
    Df Model:                           3                                         
    Covariance Type:            nonrobust                                         
    =============================================================================================
                                    coef    std err          t      P>|t|      [95.0% Conf. Int.]
    ---------------------------------------------------------------------------------------------
    Intercept                    28.6500     13.111      2.185      0.032         2.536    54.764
    C(bin_polityscore)[T.1.0]    18.1109     13.670      1.325      0.189        -9.114    45.336
    C(bin_polityscore)[T.2.0]    19.9574     13.588      1.469      0.146        -7.106    47.021
    C(bin_polityscore)[T.3.0]    22.8464     13.572      1.683      0.096        -4.184    49.877
    ==============================================================================
    Omnibus:                        4.114   Durbin-Watson:                   1.774
    Prob(Omnibus):                  0.128   Jarque-Bera (JB):                2.074
    Skew:                           0.009   Prob(JB):                        0.354
    Kurtosis:                       2.211   Cond. No.                         14.6
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    

As can be seen from above, the p-value [_Prob (F-statistic)_: 0.365] is greater than 0.05, hence we **fail to reject** the NULL Hypothesis. 



Since the statistical test was not significant, we need **not** perform the POST HOC test.


    
