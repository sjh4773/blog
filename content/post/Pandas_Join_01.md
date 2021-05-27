---
title: join 함수
date: 2021-04-11 21:52:29
categories:
  - pandas 공부
tags:
  - pandas 공부
---

# JOIN 함수

Key Value를 기준으로 데이터 프레임을 병합하는 함수

![](/image/join.PNG)


```python
import pandas as pd

```


```python
d1 = {'Asset_Allocation':[1,2,3,4,5,6],
  'stock':['IDEXX','Zoetis','Freshpet','Chewy','Trupanion','WOOF']
     }
df1 = pd.DataFrame(d1)
 
d2 = {'Asset_Allocation':[2,3,6,8],
    'Analyze':['Buy','Hold','Sell', 'None']}
df2 = pd.DataFrame(d2)
```


```python
df1
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
      <th>Asset_Allocation</th>
      <th>stock</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>IDEXX</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Zoetis</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Freshpet</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Chewy</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Trupanion</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>WOOF</td>
    </tr>
  </tbody>
</table>
</div>




```python
df2
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
      <th>Asset_Allocation</th>
      <th>Analyze</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Buy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>Hold</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>Sell</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



#### Inner 조인 수행. 


```python
inner_join_result = pd.merge(df1, df2, on='Asset_Allocation', how='inner')
inner_join_result
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
      <th>Asset_Allocation</th>
      <th>stock</th>
      <th>Analyze</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Zoetis</td>
      <td>Buy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>Freshpet</td>
      <td>Hold</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>WOOF</td>
      <td>Sell</td>
    </tr>
  </tbody>
</table>
</div>




```python
inner_join_result = df1.merge(df2, on='Asset_Allocation', how='inner')
inner_join_result
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
      <th>Asset_Allocation</th>
      <th>stock</th>
      <th>Analyze</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Zoetis</td>
      <td>Buy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>Freshpet</td>
      <td>Hold</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>WOOF</td>
      <td>Sell</td>
    </tr>
  </tbody>
</table>
</div>



#### Left Outer 조인 수행.


```python
left_outer_join_result = df1.merge(df2, on='Asset_Allocation', how='left')
left_outer_join_result
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
      <th>Asset_Allocation</th>
      <th>stock</th>
      <th>Analyze</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>IDEXX</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Zoetis</td>
      <td>Buy</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Freshpet</td>
      <td>Hold</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Chewy</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Trupanion</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>WOOF</td>
      <td>Sell</td>
    </tr>
  </tbody>
</table>
</div>



#### Right Outer 조인 수행. 


```python
right_outer_join_result = df1.merge(df2, on='Asset_Allocation', how='right')
right_outer_join_result
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
      <th>Asset_Allocation</th>
      <th>stock</th>
      <th>Analyze</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Zoetis</td>
      <td>Buy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>Freshpet</td>
      <td>Hold</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>WOOF</td>
      <td>Sell</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8</td>
      <td>NaN</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
left_outer_join_result_01 = df2.merge(df1, on='Asset_Allocation', how='left')
left_outer_join_result_01
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
      <th>Asset_Allocation</th>
      <th>Analyze</th>
      <th>stock</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Buy</td>
      <td>Zoetis</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>Hold</td>
      <td>Freshpet</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>Sell</td>
      <td>WOOF</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



#### Full outer join 수행.


```python
full_outer_join_result = pd.merge(df1, df2, on='Asset_Allocation', how='outer')
full_outer_join_result
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
      <th>Asset_Allocation</th>
      <th>stock</th>
      <th>Analyze</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>IDEXX</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Zoetis</td>
      <td>Buy</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Freshpet</td>
      <td>Hold</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Chewy</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Trupanion</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>WOOF</td>
      <td>Sell</td>
    </tr>
    <tr>
      <th>6</th>
      <td>8</td>
      <td>NaN</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
