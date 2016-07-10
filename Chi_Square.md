
# Determine if a country's life expectancy is affected by the polity score of that country, i.e. is life expectancy independent or dependent on the polity score.

## Data

Data for this study comes from the Gapminder World Dataset collected by the Gapminder Foundation. The Gapminder World Dataset contains data collected from more than 200 countries/areas for more 500 variables.


## Description of Variables

Below is the description of the variables

1. **Democracy Score** (variable code: polityscore, Unit: Scale -10 being lowest, 10 highest) - Score from -10 to 10.
2. **Life Expectancy** (variable code: lifeexpectancy, Unit: Number of years) - No of years of life expectancy (at birth, provided mortality patterns remain constant).

**Life Expectancy** is the response variable and **Democracy Score** is the primary explanatory variable.

### Import the libraries and loading the data into a pandas dataframe


    import numpy as np
    import pandas as pd
    import scipy.stats
    
    # bug fix for display formats to avoid run time errors
    pd.set_option('display.float_format', lambda x:'%.3f'%x)
    
    # Loading the data 
    data = pd.read_csv('gapminder.csv')

### Converting variables to numeric


    # convert variables to numeric format using convert_objects function
    data['polityscore'] = pd.to_numeric(data['polityscore'], errors='coerce')
    data['lifeexpectancy'] = pd.to_numeric(data['lifeexpectancy'], errors='coerce')
    

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
      <td>25.6000003814697</td>
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
      <td>42.0999984741211</td>
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
      <td>31.7000007629394</td>
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
      <td></td>
      <td></td>
      <td>81</td>
      <td>nan</td>
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
      <td>69.4000015258789</td>
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



## Binning the variables

Since we are doing an Chi Square test of Independence, both the response variable and the explanatory variable have to be categorical.


_Binning (categorizing) the explanatory variable_:

Here the explanatory variable **Democracy Score** has values from -10 to 10. We would categorize it into four categories.

Democracy Score between -10 and -6 (both inclusive) : Categorized as 0 (_Authoritarian Regimes_)

Democracy Score between  -5 and -1 (both inclusive) : Categorized as 1 (_Hybrid Regimes_)

Democracy Score between   0 and  5 (both inclusive) : Categorized as 2 (_Flawed Democracies_)

Democracy Score between   6 and 10 (both inclusive) : Categorized as 3 (_Full Democracies_)

Ideas for this classification have been taken from the Democracy Index entry in Wikipedia available [here](https://en.wikipedia.org/wiki/Democracy_Index).




    ## Binning the explanatory variable
    
    def categorize_dem_score(x):
        if x['polityscore'] >= -10 and x['polityscore'] <= -6:
            return 0
        elif x['polityscore'] >= -5 and x['polityscore'] <= -1:
            return 1
        elif x['polityscore'] >= 0 and x['polityscore'] <= 5:
            return 2
        elif x['polityscore'] >= 6 and x['polityscore'] <= 10:
            return 3
        
    data['bin_polityscore'] = data.apply(lambda x: categorize_dem_score(x), axis =1)
    

_Binning (categorizing) the response variable_:

As per WHO data available [here](https://en.wikipedia.org/wiki/List_of_countries_by_life_expectancy),  the average life expectancy at birth globally was 71.0 years. We would be classifying countries with life expectancy as 71 and less as "Low life expectancy" countries and those with more than 71 as High life expectancy

Here the response variable **Life Expectancy** has values between 40 and 90. We would categorize it into two categories.

Life Expectancy from 40 years to 71 years (both inclusive) : Categorized as 0 (Low life expectancy)

Life Expectancy from 72 years to 90 years (both inclusive) : Categorized as 1 (High life expectancy)




    ## Binning the response variable
    
    def categorize_life_expect(x):
        if   x['lifeexpectancy'] <= 71:
            return 0
        else:
            return 1
        
    data['bin_lifeexpectancy'] = data.apply(lambda x: categorize_life_expect(x), axis =1)

### Choosing the required variables

Selecting only the **Democracy Score** and **Life Expectancy** from the data set and dropping the NA values.


    data_complete = data[['bin_polityscore','bin_lifeexpectancy']].dropna()

### View of the data now


    data_complete.head()




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bin_polityscore</th>
      <th>bin_lifeexpectancy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.000</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3.000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.000</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>3.000</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



### Chi Square Test of Indepedence

### Hypothesis Testing

**Null Hypothesis**: There is no difference in the life expectancy rate between countries having different levels of polity score.

**Alternative Hypothesis**: There is difference in the life expectancy rate between countries having different levels of polity score.


###Generating the Contigency Table


    contingency_tab = pd.crosstab(index=data_complete['bin_lifeexpectancy'],columns=data_complete['bin_polityscore'])
    contingency_tab.index= ["Low Life Expectancy","High Life Expectancy"]
    contingency_tab




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>bin_polityscore</th>
      <th>0.0</th>
      <th>1.0</th>
      <th>2.0</th>
      <th>3.0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low Life Expectancy</th>
      <td>10</td>
      <td>18</td>
      <td>21</td>
      <td>28</td>
    </tr>
    <tr>
      <th>High Life Expectancy</th>
      <td>13</td>
      <td>5</td>
      <td>4</td>
      <td>62</td>
    </tr>
  </tbody>
</table>
</div>



###Chi Square Test of Independence


    print('chi-square value, p value, expected counts')
    chi2= scipy.stats.chi2_contingency(contingency_tab)
    chi2

    chi-square value, p value, expected counts
    




    (31.89952861952861,
     5.4949388506609752e-07,
     3,
     array([[ 11.        ,  11.        ,  11.95652174,  43.04347826],
            [ 12.        ,  12.        ,  13.04347826,  46.95652174]]))



The **p-value** is 5.4949388506609752e-07 (< 0.05), which concludes that country's life expectancy and the polity score are significantly associated. They are not independent. So we can **reject** the NULL hypothesis.

## POST HOC Test

Since the explanatory variable has more than 2 levels and the statistical test is significant, we need to perform POST HOC test.

### Bonferroni Adjustment

We would need to perform 6 comparisons for the four level of explanatory variable. 

Hence the adjusted p-value due to Bonferroni Adjustment would be = 0.05/6 = 0.0083. 

We would need to compare against this **adjusted p-value** of **0.0083**.

### Comparison 1

Between Democracy Score 0 and 1


    recode1 = {0:0, 1:1}
    data_complete['COMP0vs1'] = data_complete['bin_polityscore'].map(recode1)
    con_tab1 = pd.crosstab(index=data_complete['bin_lifeexpectancy'], columns=data_complete['COMP0vs1'])
    con_tab1.index= ["Low Life Expectancy","High Life Expectancy"]
    con_tab1




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>COMP0vs1</th>
      <th>0.0</th>
      <th>1.0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low Life Expectancy</th>
      <td>10</td>
      <td>18</td>
    </tr>
    <tr>
      <th>High Life Expectancy</th>
      <td>13</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




    print('Comparison 1: chi-square value, p value, expected counts')
    chi_sq_1= scipy.stats.chi2_contingency(con_tab1)
    chi_sq_1

    Comparison 1: chi-square value, p value, expected counts
    




    (4.4722222222222223, 0.034450158377685704, 1, array([[ 14.,  14.],
            [  9.,   9.]]))



###Comparison 2

Between Democracy Score 0 and 2


    recode2 = {0:0, 2:2}
    data_complete['COMP0vs2'] = data_complete['bin_polityscore'].map(recode2)
    con_tab2 = pd.crosstab(index=data_complete['bin_lifeexpectancy'], columns=data_complete['COMP0vs2'])
    con_tab2.index= ["Low Life Expectancy","High Life Expectancy"]
    con_tab2




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>COMP0vs2</th>
      <th>0.0</th>
      <th>2.0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low Life Expectancy</th>
      <td>10</td>
      <td>21</td>
    </tr>
    <tr>
      <th>High Life Expectancy</th>
      <td>13</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




    print('Comparison 2: chi-square value, p value, expected counts')
    chi_sq_2= scipy.stats.chi2_contingency(con_tab2)
    chi_sq_2

    Comparison 2: chi-square value, p value, expected counts
    




    (6.9191914858510017,
     0.0085275522655948802,
     1,
     array([[ 14.85416667,  16.14583333],
            [  8.14583333,   8.85416667]]))



###Comparison 3

Between Democracy Score 0 and 3


    recode3 = {0:0, 3:3}
    data_complete['COMP0vs3'] = data_complete['bin_polityscore'].map(recode3)
    con_tab3 = pd.crosstab(index=data_complete['bin_lifeexpectancy'], columns=data_complete['COMP0vs3'])
    con_tab3.index= ["Low Life Expectancy","High Life Expectancy"]
    con_tab3




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>COMP0vs3</th>
      <th>0.0</th>
      <th>3.0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low Life Expectancy</th>
      <td>10</td>
      <td>28</td>
    </tr>
    <tr>
      <th>High Life Expectancy</th>
      <td>13</td>
      <td>62</td>
    </tr>
  </tbody>
</table>
</div>




    print('Comparison 3: chi-square value, p value, expected counts')
    chi_sq_3= scipy.stats.chi2_contingency(con_tab3)
    chi_sq_3

    Comparison 3: chi-square value, p value, expected counts
    




    (0.76234057971014513,
     0.38259703536999212,
     1,
     array([[  7.73451327,  30.26548673],
            [ 15.26548673,  59.73451327]]))



###Comparison 4

Between Democracy Score 1 and 2


    recode4 = {1:1, 2:2}
    data_complete['COMP1vs2'] = data_complete['bin_polityscore'].map(recode4)
    con_tab4 = pd.crosstab(index=data_complete['bin_lifeexpectancy'], columns=data_complete['COMP1vs2'])
    con_tab4.index= ["Low Life Expectancy","High Life Expectancy"]
    con_tab4




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>COMP1vs2</th>
      <th>1.0</th>
      <th>2.0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low Life Expectancy</th>
      <td>18</td>
      <td>21</td>
    </tr>
    <tr>
      <th>High Life Expectancy</th>
      <td>5</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




    print('Comparison 4: chi-square value, p value, expected counts')
    chi_sq_4= scipy.stats.chi2_contingency(con_tab4)
    chi_sq_4

    Comparison 4: chi-square value, p value, expected counts
    




    (0.019264214046822742, 0.88961168392653067, 1, array([[ 18.6875,  20.3125],
            [  4.3125,   4.6875]]))



###Comparison 5

Between Democracy Score 1 and 3



    recode5 = {1:1, 3:3}
    data_complete['COMP1vs3'] = data_complete['bin_polityscore'].map(recode5)
    con_tab5 = pd.crosstab(index=data_complete['bin_lifeexpectancy'], columns=data_complete['COMP1vs3'])
    con_tab5.index= ["Low Life Expectancy","High Life Expectancy"]
    con_tab5




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>COMP1vs3</th>
      <th>1.0</th>
      <th>3.0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low Life Expectancy</th>
      <td>18</td>
      <td>28</td>
    </tr>
    <tr>
      <th>High Life Expectancy</th>
      <td>5</td>
      <td>62</td>
    </tr>
  </tbody>
</table>
</div>




    print('Comparison 5: chi-square value, p value, expected counts')
    chi_sq_5= scipy.stats.chi2_contingency(con_tab5)
    chi_sq_5

    Comparison 5: chi-square value, p value, expected counts
    




    (14.975417219197018,
     0.00010892091021745052,
     1,
     array([[  9.36283186,  36.63716814],
            [ 13.63716814,  53.36283186]]))



###Comparison 6

Between Democracy Score 2 and 3


    recode6 = {2:2, 3:3}
    data_complete['COMP2vs3'] = data_complete['bin_polityscore'].map(recode6)
    con_tab6 = pd.crosstab(index=data_complete['bin_lifeexpectancy'], columns=data_complete['COMP2vs3'])
    con_tab6.index= ["Low Life Expectancy","High Life Expectancy"]
    con_tab6




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>COMP2vs3</th>
      <th>2.0</th>
      <th>3.0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low Life Expectancy</th>
      <td>21</td>
      <td>28</td>
    </tr>
    <tr>
      <th>High Life Expectancy</th>
      <td>4</td>
      <td>62</td>
    </tr>
  </tbody>
</table>
</div>




    print('Comparison 6: chi-square value, p value, expected counts')
    chi_sq_6= scipy.stats.chi2_contingency(con_tab6)
    chi_sq_6

    Comparison 6: chi-square value, p value, expected counts
    




    (20.269905689548548,
     6.7250103253638956e-06,
     1,
     array([[ 10.65217391,  38.34782609],
            [ 14.34782609,  51.65217391]]))



The p-values for **Comparison 5** (Between Democracy Score 1 and 3 : 0.0001) and **Comparison 6** (Between Democracy Score 2 and 3 : 6.72e-06) are less than the adjusted p-value due of 0.0083.

Thus countries with Democracy score of 3 is **significantly different** from countries with Democracy score of 1 and 2.


    
