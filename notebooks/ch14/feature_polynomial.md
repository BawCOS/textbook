
<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Predicting-Ice-Cream-Ratings" data-toc-modified-id="Predicting-Ice-Cream-Ratings-1">Predicting Ice Cream Ratings</a></span></li><li><span><a href="#Polynomial-Features" data-toc-modified-id="Polynomial-Features-2">Polynomial Features</a></span></li><li><span><a href="#Polynomial-Regression" data-toc-modified-id="Polynomial-Regression-3">Polynomial Regression</a></span></li><li><span><a href="#Increasing-the-Degree" data-toc-modified-id="Increasing-the-Degree-4">Increasing the Degree</a></span></li><li><span><a href="#Summary" data-toc-modified-id="Summary-5">Summary</a></span></li></ul></div>


```python
# HIDDEN
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
%matplotlib inline
import ipywidgets as widgets
from ipywidgets import interact, interactive, fixed, interact_manual
import nbinteract as nbi

sns.set()
sns.set_context('talk')
np.set_printoptions(threshold=20, precision=2, suppress=True)
pd.options.display.max_rows = 7
pd.options.display.max_columns = 8
```


```python
# HIDDEN
def df_interact(df, nrows=7, ncols=7):
    '''
    Outputs sliders that show rows and columns of df
    '''
    def peek(row=0, col=0):
        return df.iloc[row:row + nrows, col:col + ncols]
    if len(df.columns) <= ncols:
        interact(peek, row=(0, len(df) - nrows, nrows), col=fixed(0))
    else:
        interact(peek,
                 row=(0, len(df) - nrows, nrows),
                 col=(0, len(df.columns) - ncols))
    print('({} rows, {} columns) total'.format(df.shape[0], df.shape[1]))
```


```python
# HIDDEN
# To determine which columns to regress
# ice_orig = pd.read_csv('icecream_orig.csv')
# cols = ['aerated', 'afterfeel', 'almond', 'buttery', 'color', 'cooling',
#        'creamy', 'doughy', 'eggy', 'fat', 'fat_level', 'fatty', 'hardness',
#        'ice_crystals', 'id', 'liking_flavor', 'liking_texture', 'melt_rate',
#        'melting_rate', 'milky', 'sugar', 'sugar_level', 'sweetness',
#        'tackiness', 'vanilla']

# melted = ice_orig.melt(id_vars='overall', value_vars=cols, var_name='type')
# sns.lmplot(x='value', y='overall', col='type', col_wrap=5, data=melted,
#            sharex=False, fit_reg=False)
```

## Predicting Ice Cream Ratings

Suppose we are trying to create new, popular ice cream flavors. We are interested in the following regression problem: given the sweetness of an ice cream flavor, predict its overall taste rating out of 7.


```python
ice = pd.read_csv('icecream.csv')
ice
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sweetness</th>
      <th>overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4.1</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6.9</td>
      <td>5.4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8.3</td>
      <td>5.8</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>11.0</td>
      <td>5.9</td>
    </tr>
    <tr>
      <th>7</th>
      <td>11.7</td>
      <td>5.5</td>
    </tr>
    <tr>
      <th>8</th>
      <td>11.9</td>
      <td>5.4</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 2 columns</p>
</div>



Although we expect that an ice cream flavor that is not sweet enough would receive a low rating, we also expect that an ice flavor that is too sweet would also receive a low rating. This is reflected in the scatter plot of overall rating and sweetness:


```python
# HIDDEN
sns.lmplot(x='sweetness', y='overall', data=ice, fit_reg=False)
plt.title('Overall taste rating vs. sweetness');
```


![png](feature_polynomial_files/feature_polynomial_7_0.png)


Unfortunately, a linear model alone cannot take this increase-then-decrease behavior into account; in a linear model, the overall rating can only increase or decrease monotonically with the sweetness. We can see that using linear regression results in a poor fit.


```python
# HIDDEN
sns.lmplot(x='sweetness', y='overall', data=ice)
plt.title('Overall taste rating vs. sweetness');
```


![png](feature_polynomial_files/feature_polynomial_9_0.png)


One useful approach for this problem is to fit a polynomial curve instead of line. Such a curve would allow us to model the fact that the overall rating increases with sweetness only up to a certain point, then decreases as sweetness increases.

With a feature engineering technique, we can simply add new columns to our data to use our linear model for polynomial regression.

## Polynomial Features

Recall that in linear regression we fit one weight for each column of our data matrix $ X $. In this case, our matrix $ X $ contains two columns: a column of all ones and the sweetness.


```python
# HIDDEN
from sklearn.preprocessing import PolynomialFeatures

first_X = PolynomialFeatures(degree=1).fit_transform(ice[['sweetness']])
pd.DataFrame(data=first_X, columns=['bias', 'sweetness'])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bias</th>
      <th>sweetness</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>4.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>6.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>8.3</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.0</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1.0</td>
      <td>11.7</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.0</td>
      <td>11.9</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 2 columns</p>
</div>



Our model is thus:

$$
f_\hat{\theta} (x) = \hat{\theta_0} + \hat{\theta_1} \cdot \text{sweetness}
$$

We can create a new column in $ X $ containing the squared values of the sweetness.


```python
# HIDDEN
second_X = PolynomialFeatures(degree=2).fit_transform(ice[['sweetness']])
pd.DataFrame(data=second_X, columns=['bias', 'sweetness', 'sweetness^2'])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bias</th>
      <th>sweetness</th>
      <th>sweetness^2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>4.1</td>
      <td>16.81</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>6.9</td>
      <td>47.61</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>8.3</td>
      <td>68.89</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.0</td>
      <td>11.0</td>
      <td>121.00</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1.0</td>
      <td>11.7</td>
      <td>136.89</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.0</td>
      <td>11.9</td>
      <td>141.61</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 3 columns</p>
</div>



Since our model learns one weight for each column of its input matrix, our model will become:

$$
f_\hat{\theta} (x) = \hat{\theta_0}
    + \hat{\theta_1} \cdot \text{sweetness}
    + \hat{\theta_2} \cdot \text{sweetness}^2
$$

Our model now fits a polynomial with degree two to our data. We can easily fit higher degree polynomials by adding columns for $ \text{sweetness}^3 $, $ \text{sweetness}^4 $, and so on.

Notice that this model is still a linear model because it is **linear in its parameters**—each $ \hat{\theta_i} $ is a scalar value of degree one. However, the model is **polynomial in its features** because its input data contains a column that is a polynomial transformation of another column.

## Polynomial Regression

To conduct polynomial regression, we use a linear model with polynomial features. Thus, we import the [`LinearRegression`](http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html#sklearn.linear_model.LinearRegression) model and [`PolynomialFeatures`](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.PolynomialFeatures.html#sklearn.preprocessing.PolynomialFeatures) transform from `scikit-learn`.


```python
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures
```

Our original data matrix $ X $ contains the following values. Remember that we include the column and row labels for reference purposes only; the actual $ X $ matrix only contains the numerical data in the table below.


```python
ice[['sweetness']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sweetness</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8.3</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>11.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>11.7</td>
    </tr>
    <tr>
      <th>8</th>
      <td>11.9</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 1 columns</p>
</div>



We first use the `PolynomialFeatures` class to transform the data, adding polynomial features of degree 2.


```python
transformer = PolynomialFeatures(degree=2)
X = transformer.fit_transform(ice[['sweetness']])
X
```




    array([[  1.  ,   4.1 ,  16.81],
           [  1.  ,   6.9 ,  47.61],
           [  1.  ,   8.3 ,  68.89],
           ...,
           [  1.  ,  11.  , 121.  ],
           [  1.  ,  11.7 , 136.89],
           [  1.  ,  11.9 , 141.61]])



Now, we fit a linear model to this data matrix.


```python
clf = LinearRegression(fit_intercept=False)
clf.fit(X, ice['overall'])
clf.coef_
```




    array([-1.3 ,  1.6 , -0.09])



The parameters above show that for this dataset, the best-fit model is:

$$
f_\hat{\theta} (x) = -1.3 + 1.6 \cdot \text{sweetness} - 0.09 \cdot \text{sweetness}^2
$$

We can now compare this model's predictions against the original data.


```python
# HIDDEN
sns.lmplot(x='sweetness', y='overall', data=ice, fit_reg=False)
xs = np.linspace(3.5, 12.5, 1000).reshape(-1, 1)
ys = clf.predict(transformer.transform(xs))
plt.plot(xs, ys)
plt.title('Degree 2 polynomial fit');
```


![png](feature_polynomial_files/feature_polynomial_27_0.png)


This model looks like a much better fit than our linear model. We can also verify that the mean squared cost for the degree 2 polynomial fit is much lower than the cost for the linear fit.


```python
# HIDDEN
y = ice['overall']
pred_linear = (
    LinearRegression(fit_intercept=False).fit(first_X, y).predict(first_X)
)
pred_quad = clf.predict(X)

def mse_cost(pred, y): return np.mean((pred - y) ** 2)

print(f'MSE cost for linear reg:     {mse_cost(pred_linear, y):.3f}')
print(f'MSE cost for deg 2 poly reg: {mse_cost(pred_quad, y):.3f}')
```

    MSE cost for linear reg:     0.323
    MSE cost for deg 2 poly reg: 0.032


## Increasing the Degree

As mentioned earlier, we are free to add higher degree polynomial features to our data. For example, we can easily create polynomial features of degree 5:


```python
# HIDDEN
second_X = PolynomialFeatures(degree=5).fit_transform(ice[['sweetness']])
pd.DataFrame(data=second_X,
             columns=['bias', 'sweetness', 'sweetness^2', 'sweetness^3',
                      'sweetness^4', 'sweetness^5'])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bias</th>
      <th>sweetness</th>
      <th>sweetness^2</th>
      <th>sweetness^3</th>
      <th>sweetness^4</th>
      <th>sweetness^5</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>4.1</td>
      <td>16.81</td>
      <td>68.921</td>
      <td>282.5761</td>
      <td>1158.56201</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>6.9</td>
      <td>47.61</td>
      <td>328.509</td>
      <td>2266.7121</td>
      <td>15640.31349</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>8.3</td>
      <td>68.89</td>
      <td>571.787</td>
      <td>4745.8321</td>
      <td>39390.40643</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.0</td>
      <td>11.0</td>
      <td>121.00</td>
      <td>1331.000</td>
      <td>14641.0000</td>
      <td>161051.00000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1.0</td>
      <td>11.7</td>
      <td>136.89</td>
      <td>1601.613</td>
      <td>18738.8721</td>
      <td>219244.80357</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.0</td>
      <td>11.9</td>
      <td>141.61</td>
      <td>1685.159</td>
      <td>20053.3921</td>
      <td>238635.36599</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 6 columns</p>
</div>



Fitting a linear model using these features results in a degree five polynomial regression.


```python
# HIDDEN
trans_five = PolynomialFeatures(degree=5)
X_five = trans_five.fit_transform(ice[['sweetness']])
clf_five = LinearRegression(fit_intercept=False).fit(X_five, y)

sns.lmplot(x='sweetness', y='overall', data=ice, fit_reg=False)
xs = np.linspace(3.5, 12.5, 1000).reshape(-1, 1)
ys = clf_five.predict(trans_five.transform(xs))
plt.plot(xs, ys)
plt.title('Degree 5 polynomial fit');
```


![png](feature_polynomial_files/feature_polynomial_33_0.png)


The plot shows that a degree five polynomial seems to fit the data roughly as well as a degree two polynomial. In fact, the mean squared cost for the degree five polynomial is almost half of the cost for the degree two polynomial.


```python
pred_five = clf_five.predict(X_five)

print(f'MSE cost for linear reg:     {mse_cost(pred_linear, y):.3f}')
print(f'MSE cost for deg 2 poly reg: {mse_cost(pred_quad, y):.3f}')
print(f'MSE cost for deg 5 poly reg: {mse_cost(pred_five, y):.3f}')
```

    MSE cost for linear reg:     0.323
    MSE cost for deg 2 poly reg: 0.032
    MSE cost for deg 5 poly reg: 0.017


This suggests that we might do even better by increasing the degree even more. Why not a degree 10 polynomial?


```python
# HIDDEN
trans_ten = PolynomialFeatures(degree=10)
X_ten = trans_ten.fit_transform(ice[['sweetness']])
clf_ten = LinearRegression(fit_intercept=False).fit(X_ten, y)

sns.lmplot(x='sweetness', y='overall', data=ice, fit_reg=False)
xs = np.linspace(3.5, 12.5, 1000).reshape(-1, 1)
ys = clf_ten.predict(trans_ten.transform(xs))
plt.plot(xs, ys)
plt.title('Degree 10 polynomial fit')
plt.ylim(3, 7);
```


![png](feature_polynomial_files/feature_polynomial_37_0.png)


Here are the mean squared costs for the regression models we've seen thus far:


```python
# HIDDEN
pred_ten = clf_ten.predict(X_ten)

print(f'MSE cost for linear reg:      {mse_cost(pred_linear, y):.3f}')
print(f'MSE cost for deg 2 poly reg:  {mse_cost(pred_quad, y):.3f}')
print(f'MSE cost for deg 5 poly reg:  {mse_cost(pred_five, y):.3f}')
print(f'MSE cost for deg 10 poly reg: {mse_cost(pred_ten, y):.3f}')
```

    MSE cost for linear reg:      0.323
    MSE cost for deg 2 poly reg:  0.032
    MSE cost for deg 5 poly reg:  0.017
    MSE cost for deg 10 poly reg: 0.000


The degree 10 polynomial has a cost of zero! This makes sense if we take a closer look at the plot; the degree ten polynomial manages to pass through the precise location of each point in the data.

However, you should feel hesitant to use the degree 10 polynomial to predict ice cream ratings. Intuitively, the degree 10 polynomial seems to fit our specific set of data too closely. If we take another set of data and plot them on the scatter plot above, we can expect that they fall close to our original set of data. When we do this, however, the degree 10 polynomial suddenly seems like a poor fit while the degree 2 polynomial still looks reasonable.


```python
# HIDDEN
# sns.lmplot(x='sweetness', y='overall', data=ice, fit_reg=False)
np.random.seed(1)
x_devs = np.random.normal(scale=0.4, size=len(ice))
y_devs = np.random.normal(scale=0.4, size=len(ice))

plt.figure(figsize=(10, 5))

# Degree 10
plt.subplot(121)
ys = clf_ten.predict(trans_ten.transform(xs))
plt.plot(xs, ys)
plt.scatter(ice['sweetness'] + x_devs,
            ice['overall'] + y_devs,
            c='g')
plt.title('Degree 10 poly, second set of data')
plt.ylim(3, 7);

plt.subplot(122)
ys = clf.predict(transformer.transform(xs))
plt.plot(xs, ys)
plt.scatter(ice['sweetness'] + x_devs,
            ice['overall'] + y_devs,
            c='g')
plt.title('Degree 2 poly, second set of data')
plt.ylim(3, 7);
```


![png](feature_polynomial_files/feature_polynomial_41_0.png)


We can see that in this case, degree two polynomial features work better than both no transformation and degree ten polynomial features.

This raises the natural question: in general, how do we determine which degree polynomial to fit? Although we are tempted to use the cost on the training dataset to pick the best polynomial, we have seen that using this cost can pick a model that is too complex. Instead, we want to evaluate our model on data that is not used to fit parameters.

## Summary

In this section, we introduce another feature engineering technique: adding polynomial features to the data in order to perform polynomial regression. Like one-hot encoding, adding polynomial features allows us to use our linear regression model effectively on more types of data.

We have also encountered a fundamental issue with feature engineering. Adding many features to the data gives the model a lower cost on its original set of data but often results in a less accurate model on new sets of data. 
