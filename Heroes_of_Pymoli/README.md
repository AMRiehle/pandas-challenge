
# Heroes of Pymoli

GWU Data Analytics Bootcamp Homework 4

### Observable Trends

* Males make up a significantly larger portion of the player base (or at least the player base who makes purchases) than women and also tend to spend more than women.

* The majority of players are between the ages of 20-24, but the typical 20-24 year old pays less than players in other age brackets - both older and younger.

* Higher-priced items tend to be more profitable even if they aren't more frequently purchased. The price difference often compensates for any differences in purchase frequency.


```python
# Load dependencies

import pandas as pd
import numpy as np

# Read in data and create initial dataframe

json = "../Resources/Heroes.JSON"
df = pd.read_json(json)
```

### Player Count


```python
# Calculate number of unique players/individuals

num_players = df['SN'].nunique()
pd.DataFrame({"Total Players":[num_players]})
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>573</td>
    </tr>
  </tbody>
</table>

### Purchasing Analysis (Total)


```python
# Calculate summary statistics for data

unique_items_count = df['Item ID'].nunique()
avg_purchase_price = df['Price'].mean()
total_purchases = df['Price'].count()
total_revenue = df['Price'].sum()

# Create and format dataframe

tot_purchases_df = pd.DataFrame({"Number of Unique Items":[unique_items_count], "Average Price":[avg_purchase_price], "Number of Purchases":[total_purchases], "Total Revenue":[total_revenue]})
tot_purchases_df = tot_purchases_df.reindex(['Number of Unique Items','Average Price','Number of Purchases','Total Revenue'], axis=1)

# Format data

tot_purchases_df['Average Price'] = tot_purchases_df['Average Price'].map("${:.2f}".format)
tot_purchases_df['Total Revenue'] = tot_purchases_df['Total Revenue'].map("${:,.2f}".format)

# Print data

tot_purchases_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number of Unique Items</th>
      <th>Average Price</th>
      <th>Number of Purchases</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>183</td>
      <td>$2.93</td>
      <td>780</td>
      <td>$2,286.33</td>
    </tr>
  </tbody>
</table>

### Gender Demographics


```python
# Isolate unique individuals

sn_unique_df = df.drop_duplicates(subset="SN")

# Group data by gender

sn_unique_gender_df = sn_unique_df.groupby(['Gender'])

# Calculate count and percent of unique individuals by gender

sn_unique_grouped_gender_count = sn_unique_gender_df[['Gender']].count()
sn_unique_grouped_gender_percent = (sn_unique_gender_df[['Gender']].count() / sn_unique_gender_df[['Gender']].count().sum()) * 100

# Combine count and percent dataframes

sn_unique_gender_demo_df = pd.merge(sn_unique_grouped_gender_percent, sn_unique_grouped_gender_count, left_index=True, right_index=True, how='outer')
sn_unique_gender_demo_df = sn_unique_gender_demo_df.rename(columns={"Gender_x":"Percent of Players", "Gender_y":"Total Count"})

# Format data

sn_unique_gender_demo_df['Percent of Players'] = sn_unique_gender_demo_df['Percent of Players'].map("{:.2f}%".format)

# Sort data by descending gender frequency

sn_unique_gender_demo_df.sort_values('Percent of Players', ascending=False)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Percent of Players</th>
      <th>Total Count</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Male</th>
      <td>81.15%</td>
      <td>465</td>
    </tr>
    <tr>
      <th>Female</th>
      <td>17.45%</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>1.40%</td>
      <td>8</td>
    </tr>
  </tbody>
</table>

### Purchasing Analysis (Gender)


```python
# Group dataframe and perform initial aggregation

grouped_gender_df = df.groupby(['Gender'])
gender_purchase_df = grouped_gender_df.agg({'Gender':['count'], 'Price':['mean', 'sum']})

# Calculate normalized totals (total purchases / unique individuals)

norm_gender_tots = grouped_gender_df['Price'].sum() / (sn_unique_gender_df['Gender'].count())
gender_purchase_df['Normalized Totals'] = norm_gender_tots

# Format dataframe and rename columns

gender_purchase_df.columns = gender_purchase_df.columns.droplevel()
gender_purchase_df = gender_purchase_df.rename(columns={"count":"Purchase Count", "mean":"Average Purchase Price", "sum":"Total Purchase Value", "":"Normalized Totals"})

# Format data

gender_purchase_df['Average Purchase Price'] = gender_purchase_df['Average Purchase Price'].map("${:.2f}".format)
gender_purchase_df['Total Purchase Value'] = gender_purchase_df['Total Purchase Value'].map("${:,.2f}".format)
gender_purchase_df['Normalized Totals'] = gender_purchase_df['Normalized Totals'].map("${:,.2f}".format)

# Print data

gender_purchase_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>136</td>
      <td>$2.82</td>
      <td>$382.91</td>
      <td>$3.83</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>633</td>
      <td>$2.95</td>
      <td>$1,867.68</td>
      <td>$4.02</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>11</td>
      <td>$3.25</td>
      <td>$35.74</td>
      <td>$4.47</td>
    </tr>
  </tbody>
</table>

### Age Demographics


```python
# Create and assign age brackets

bins = [0, 9, 14, 19, 24, 29, 34, 39, 44, 49]
group_names = ['Under 10', '10-14', '15-19', '20-24', '25-29', '30-34', '35-39', '40-44', '45-49']
df['Age Bracket'] = pd.cut(df["Age"], bins, labels=group_names)

# Isolate unique players 

sn_unique_df = df.drop_duplicates(subset="SN")

# Calculate count and percent of unique players in dataframe

sn_unique_age_df = sn_unique_df.groupby(['Age Bracket'])
sn_unique_grouped_age_count = sn_unique_age_df[['Age']].count()
sn_unique_grouped_age_percent = (sn_unique_age_df[['Age']].count() / sn_unique_age_df[['Age']].count().sum()) * 100

# Combine count and percent dataframes

sn_unique_age_demo_df = pd.merge(sn_unique_grouped_age_percent, sn_unique_grouped_age_count, left_index=True, right_index=True, how='outer')

# Rename dataframe columns and format data

sn_unique_age_demo_df = sn_unique_age_demo_df.rename(columns={"Age_x":"Percent of Players", "Age_y":"Total Count"})
sn_unique_age_demo_df['Percent of Players'] = sn_unique_age_demo_df['Percent of Players'].map("{:.2f}%".format)

# Print data

sn_unique_age_demo_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Percent of Players</th>
      <th>Total Count</th>
    </tr>
    <tr>
      <th>Age Bracket</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Under 10</th>
      <td>3.32%</td>
      <td>19</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>4.01%</td>
      <td>23</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>17.45%</td>
      <td>100</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>45.20%</td>
      <td>259</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>15.18%</td>
      <td>87</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>8.20%</td>
      <td>47</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>4.71%</td>
      <td>27</td>
    </tr>
    <tr>
      <th>40-44</th>
      <td>1.75%</td>
      <td>10</td>
    </tr>
    <tr>
      <th>45-49</th>
      <td>0.17%</td>
      <td>1</td>
    </tr>
  </tbody>
</table>

### Purchasing Analysis (Age)


```python
# Group and aggregate data

grouped_age_df = df.groupby(['Age Bracket'])
age_purchase_df = df.groupby(['Age Bracket']).agg({'Age':['count'], 'Price':['mean', 'sum']})

# Calculate normalized totals (total purchases / unique individuals)

age_purchase_df['Normalized Totals'] = grouped_age_df['Price'].sum() / sn_unique_age_df['Age'].count()

# Format dataframe

age_purchase_df.columns = age_purchase_df.columns.droplevel()
age_purchase_df = age_purchase_df.rename(columns={"count":"Purchase Count", "mean":"Average Purchase Price", "sum":"Total Purchase Value", "":"Normalized Totals"})

# Format data

age_purchase_df['Average Purchase Price'] = age_purchase_df['Average Purchase Price'].map("${:.2f}".format)
age_purchase_df['Total Purchase Value'] = age_purchase_df['Total Purchase Value'].map("${:,.2f}".format)
age_purchase_df['Normalized Totals'] = age_purchase_df['Normalized Totals'].map("${:,.2f}".format)

# Print data

age_purchase_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Age Bracket</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Under 10</th>
      <td>28</td>
      <td>$2.98</td>
      <td>$83.46</td>
      <td>$4.39</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>35</td>
      <td>$2.77</td>
      <td>$96.95</td>
      <td>$4.22</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>133</td>
      <td>$2.91</td>
      <td>$386.42</td>
      <td>$3.86</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>336</td>
      <td>$2.91</td>
      <td>$978.77</td>
      <td>$3.78</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>125</td>
      <td>$2.96</td>
      <td>$370.33</td>
      <td>$4.26</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>64</td>
      <td>$3.08</td>
      <td>$197.25</td>
      <td>$4.20</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>42</td>
      <td>$2.84</td>
      <td>$119.40</td>
      <td>$4.42</td>
    </tr>
    <tr>
      <th>40-44</th>
      <td>16</td>
      <td>$3.19</td>
      <td>$51.03</td>
      <td>$5.10</td>
    </tr>
    <tr>
      <th>45-49</th>
      <td>1</td>
      <td>$2.72</td>
      <td>$2.72</td>
      <td>$2.72</td>
    </tr>
  </tbody>
</table>

### Top Spenders


```python
# Aggregate data and format dataframe

top_spenders_df = df.groupby(['SN']).agg({'Price':['count', 'mean', 'sum']})
top_spenders_df.columns = top_spenders_df.columns.droplevel()
top_spenders_df = top_spenders_df.rename(columns={"count":"Purchase Count", "mean":"Average Purchase Price", "sum":"Total Purchase Value"})

# Sort data and grab first 5 rows

top_spenders_df = top_spenders_df.sort_values('Total Purchase Value', ascending=False)
top_spenders_df = top_spenders_df.head()

# Format data

top_spenders_df['Average Purchase Price'] = top_spenders_df['Average Purchase Price'].map("${:,.2f}".format)
top_spenders_df['Total Purchase Value'] = top_spenders_df['Total Purchase Value'].map("${:,.2f}".format)

# Print data

top_spenders_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Undirrala66</th>
      <td>5</td>
      <td>$3.41</td>
      <td>$17.06</td>
    </tr>
    <tr>
      <th>Saedue76</th>
      <td>4</td>
      <td>$3.39</td>
      <td>$13.56</td>
    </tr>
    <tr>
      <th>Mindimnya67</th>
      <td>4</td>
      <td>$3.18</td>
      <td>$12.74</td>
    </tr>
    <tr>
      <th>Haellysu29</th>
      <td>3</td>
      <td>$4.24</td>
      <td>$12.73</td>
    </tr>
    <tr>
      <th>Eoda93</th>
      <td>3</td>
      <td>$3.86</td>
      <td>$11.58</td>
    </tr>
  </tbody>
</table>

### Most Popular Items


```python
# Aggregate data and format dataframe

grouped_items_df = df.groupby(['Item ID', 'Item Name']).agg({'Price':['count', 'mean', 'sum']})
grouped_items_df.columns = grouped_items_df.columns.droplevel()
grouped_items_df = grouped_items_df.rename(columns={"count":"Purchase Count", "mean":"Average Purchase Price", "sum":"Total Purchase Value"})

# Sort data and grab first 5 rows

pop_items_df = grouped_items_df.sort_values('Purchase Count', ascending=False)
pop_items_df = pop_items_df.head()

# Format data

pop_items_df['Average Purchase Price'] = pop_items_df['Average Purchase Price'].map("${:,.2f}".format)
pop_items_df['Total Purchase Value'] = pop_items_df['Total Purchase Value'].map("${:,.2f}".format)

# Print data

pop_items_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>39</th>
      <th>Betrayal, Whisper of Grieving Widows</th>
      <td>11</td>
      <td>$2.35</td>
      <td>$25.85</td>
    </tr>
    <tr>
      <th>84</th>
      <th>Arcane Gem</th>
      <td>11</td>
      <td>$2.23</td>
      <td>$24.53</td>
    </tr>
    <tr>
      <th>31</th>
      <th>Trickster</th>
      <td>9</td>
      <td>$2.07</td>
      <td>$18.63</td>
    </tr>
    <tr>
      <th>175</th>
      <th>Woeful Adamantite Claymore</th>
      <td>9</td>
      <td>$1.24</td>
      <td>$11.16</td>
    </tr>
    <tr>
      <th>13</th>
      <th>Serenity</th>
      <td>9</td>
      <td>$1.49</td>
      <td>$13.41</td>
    </tr>
  </tbody>
</table>

### Most Profitable Items


```python
# Aggregate data and format dataframe

grouped_items_df = df.groupby(['Item ID', 'Item Name']).agg({'Price':['count', 'mean', 'sum']})
grouped_items_df.columns = grouped_items_df.columns.droplevel()
grouped_items_df = grouped_items_df.rename(columns={"count":"Purchase Count", "mean":"Average Purchase Price", "sum":"Total Purchase Value"})

# Sort data and grab first 5 rows

rev_items_df = grouped_items_df.sort_values('Total Purchase Value', ascending=False)
rev_items_df = rev_items_df.head()

# Format data

rev_items_df['Average Purchase Price'] = rev_items_df['Average Purchase Price'].map("${:,.2f}".format)
rev_items_df['Total Purchase Value'] = rev_items_df['Total Purchase Value'].map("${:,.2f}".format)

# Print data

rev_items_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>34</th>
      <th>Retribution Axe</th>
      <td>9</td>
      <td>$4.14</td>
      <td>$37.26</td>
    </tr>
    <tr>
      <th>115</th>
      <th>Spectral Diamond Doomblade</th>
      <td>7</td>
      <td>$4.25</td>
      <td>$29.75</td>
    </tr>
    <tr>
      <th>32</th>
      <th>Orenmir</th>
      <td>6</td>
      <td>$4.95</td>
      <td>$29.70</td>
    </tr>
    <tr>
      <th>103</th>
      <th>Singed Scalpel</th>
      <td>6</td>
      <td>$4.87</td>
      <td>$29.22</td>
    </tr>
    <tr>
      <th>107</th>
      <th>Splitter, Foe Of Subtlety</th>
      <td>8</td>
      <td>$3.61</td>
      <td>$28.88</td>
    </tr>
  </tbody>
</table>