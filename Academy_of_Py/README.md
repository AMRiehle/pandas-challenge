
# Academy of Py
GWU Data Analytics Bootcamp Homework 4

### Observable Trends

* Students tend to score higher and are more likely to pass in Reading than in Math
* Students who attend Charter schools tend to score higher and are more likely to pass than students who attend District Schools
* Students who attend small schools tend to score higher and are more likely to pass than students who attend larger schools


```python
# Load dependencies

import pandas as pd
import numpy as np
```


```python
# Find data

school_csv = "../Resources/schools_complete.csv"
student_csv = "../Resources/students_complete.csv"
```


```python
# Read in data and create dataframes

school_df = pd.read_csv(school_csv)
student_df = pd.read_csv(student_csv)
```


```python
# Merge initial dataframes

school_df = school_df.rename(columns={"name":"school"})
df = pd.merge(school_df, student_df, on="school")
```

### District Summary


```python
# Set baseline for minimum passing grade

min_passing_grade = 70

# Calculate summary statistics (Overall Passing Rate is calculated as average of Math and Reading passing rates)

num_of_schools = df['School ID'].nunique()
num_of_students = df['Student ID'].nunique()
tot_school_budget = school_df['budget'].sum()
avg_math_score_tot = df['math_score'].mean()
avg_reading_score_tot = df['reading_score'].mean()
percent_passing_math_tot = df.loc[df['math_score'] >= min_passing_grade, :]['math_score'].count() / num_of_students
percent_passing_reading_tot = df.loc[df['reading_score'] >= min_passing_grade, :]['reading_score'].count() / num_of_students
overall_passing_score_tot = (percent_passing_math_tot + percent_passing_reading_tot) / 2

# Create and format summary statistics dataframe

district_summary_df = pd.DataFrame({"Total Schools":[num_of_schools], "Total Students":[num_of_students], "Total Budget":[tot_school_budget], "Average Math Score":[avg_math_score_tot], "Average Reading Score":[avg_reading_score_tot], "% Passing Math":[percent_passing_math_tot], "% Passing Reading":[percent_passing_reading_tot], "% Overall Passing Rate":[overall_passing_score_tot]})
district_summary_df = district_summary_df[['Total Schools', 'Total Students', 'Total Budget', 'Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]

# Format data

district_summary_df['Total Budget'] = district_summary_df['Total Budget'].map("${:,}".format)
district_summary_df['Average Math Score'] = district_summary_df['Average Math Score'].map("{:.2f}".format)
district_summary_df['Average Reading Score'] = district_summary_df['Average Reading Score'].map("{:.2f}".format)
district_summary_df['% Passing Math'] = (district_summary_df['% Passing Math'] * 100).map("{:.2f}%".format)
district_summary_df['% Passing Reading'] = (district_summary_df['% Passing Reading'] * 100).map("{:.2f}%".format)
district_summary_df['% Overall Passing Rate'] = (district_summary_df['% Overall Passing Rate'] * 100).map("{:.2f}%".format)

# Print data

district_summary_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39170</td>
      <td>$24,649,428</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>74.98%</td>
      <td>85.81%</td>
      <td>80.39%</td>
    </tr>
  </tbody>
</table>

### School Summary


```python
# Group and aggregate data

grouped_schools_funcs = df.groupby(['school']).agg({'Student ID':['count'], 'reading_score':['mean'], "math_score":['mean']})
grouped_schools_funcs = grouped_schools_funcs.reset_index()
grouped_schools_funcs.columns = ['School Name', 'Number of Students', 'Avg Reading Score', 'Avg Math Score']

# Add additional columns and calculate Per Student Budget

short_school_df = school_df[['school', 'type', 'budget']]
short_school_df.columns = ['School Name', 'School Type', 'Budget']
school_summary_df = pd.merge(short_school_df, grouped_schools_funcs, on='School Name')
school_summary_df['Per Student Budget'] = school_summary_df['Budget'] / school_summary_df['Number of Students']

# Calculate Percent Passing Reading

df_passing_reading = df.loc[df['reading_score'] >= min_passing_grade, :]
df_passing_reading_by_school = df_passing_reading.groupby('school')
percent_passing_reading_by_school = df_passing_reading_by_school['reading_score'].count() / df.groupby('school')['reading_score'].count()
percent_passing_reading_by_school_df = percent_passing_reading_by_school.reset_index()
percent_passing_reading_by_school_df.columns = ['School Name', '% Passing Reading']

# Calculate Percent Passing Math

df_passing_math = df.loc[df['math_score'] >= min_passing_grade, :]
df_passing_math_by_school = df_passing_math.groupby('school')
percent_passing_math_by_school = df_passing_math_by_school['math_score'].count() / df.groupby('school')['math_score'].count()
percent_passing_math_by_school_df = percent_passing_math_by_school.reset_index()
percent_passing_math_by_school_df.columns = ['School Name', '% Passing Math']

# Combine dataframes and calculate Overall Passing Rate

percent_passing_by_school_df = pd.merge(percent_passing_reading_by_school_df, percent_passing_math_by_school_df, on="School Name")
merged_school_summary_df = pd.merge(school_summary_df, percent_passing_by_school_df, on="School Name")
merged_school_summary_df['% Overall Passing Rate'] = (merged_school_summary_df['% Passing Reading'] + merged_school_summary_df['% Passing Math']) / 2

# Format dataframe

merged_school_summary_df = merged_school_summary_df.set_index('School Name')
merged_school_summary_df = merged_school_summary_df[['School Type', 'Number of Students', 'Budget', 'Per Student Budget', 'Avg Math Score', 'Avg Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]
merged_school_summary_df.columns = ['School Type', 'Total Students', 'Total School Budget', 'Per Student Budget', 'Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']

# Format data

merged_school_summary_df['Total School Budget'] = merged_school_summary_df['Total School Budget'].map("${:,}".format)
merged_school_summary_df['Per Student Budget'] = merged_school_summary_df['Per Student Budget'].map("${:,.0f}".format)
merged_school_summary_df['Average Math Score'] = merged_school_summary_df['Average Math Score'].map("{:.2f}".format)
merged_school_summary_df['Average Reading Score'] = merged_school_summary_df['Average Reading Score'].map("{:.2f}".format)
merged_school_summary_df['% Passing Math'] = (merged_school_summary_df['% Passing Math'] * 100).map("{:.2f}%".format)
merged_school_summary_df['% Passing Reading'] = (merged_school_summary_df['% Passing Reading'] * 100).map("{:.2f}%".format)
merged_school_summary_df['% Overall Passing Rate'] = (merged_school_summary_df['% Overall Passing Rate'] * 100).map("{:.2f}%".format)

# Print data

merged_school_summary_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635</td>
      <td>$655</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411</td>
      <td>$639</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600</td>
      <td>$600</td>
      <td>83.36</td>
      <td>83.73</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020</td>
      <td>$652</td>
      <td>77.29</td>
      <td>80.93</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500</td>
      <td>$625</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574</td>
      <td>$578</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356</td>
      <td>$582</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928</td>
      <td>$628</td>
      <td>77.05</td>
      <td>81.03</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087</td>
      <td>$581</td>
      <td>83.80</td>
      <td>83.81</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400</td>
      <td>$583</td>
      <td>83.68</td>
      <td>83.95</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363</td>
      <td>$637</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650</td>
      <td>$650</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916</td>
      <td>$644</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130</td>
      <td>$638</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
  </tbody>
</table>

### Top Performing Schools (By Passing Rate)


```python
# Isolate Top 5 schools by Overall Passing Rate

top_schools = merged_school_summary_df.sort_values('% Overall Passing Rate', ascending=False).head()
top_schools = top_schools[['School Type', 'Total Students', 'Total School Budget', 'Per Student Budget', 'Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]
top_schools
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356</td>
      <td>$582</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130</td>
      <td>$638</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500</td>
      <td>$625</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574</td>
      <td>$578</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
  </tbody>
</table>

### Bottom Performing Schools (By Passing Rate)


```python
# Isolate Bottom 5 schools by Overall Passing Rate

bottom_schools = merged_school_summary_df.sort_values('% Overall Passing Rate').head()
bottom_schools = bottom_schools[['School Type', 'Total Students', 'Total School Budget', 'Per Student Budget', 'Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]
bottom_schools
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363</td>
      <td>$637</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411</td>
      <td>$639</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635</td>
      <td>$655</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650</td>
      <td>$650</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916</td>
      <td>$644</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
  </tbody>
</table>

### Math Scores by Grade


```python
# Filter data by grade level

grade_9_df = df.loc[df['grade'] == '9th', :]
grade_10_df = df.loc[df['grade'] == '10th', :]
grade_11_df = df.loc[df['grade'] == '11th', :]
grade_12_df = df.loc[df['grade'] == '12th', :]

# Calculate Average Math Scores

math_by_grade_df = grade_9_df.groupby(['school']).agg({'math_score':['mean']})
math_by_grade_df['10th'] = grade_10_df.groupby(['school']).agg({'math_score':['mean']})
math_by_grade_df['11th'] = grade_11_df.groupby(['school']).agg({'math_score':['mean']})
math_by_grade_df['12th'] = grade_12_df.groupby(['school']).agg({'math_score':['mean']})

# Format dataframe

math_by_grade_df.columns = ['9th', '10th', '11th', '12th']

# Format data

math_by_grade_df['9th'] = math_by_grade_df['9th'].map("{:.2f}".format)
math_by_grade_df['10th'] = math_by_grade_df['10th'].map("{:.2f}".format)
math_by_grade_df['11th'] = math_by_grade_df['11th'].map("{:.2f}".format)
math_by_grade_df['12th'] = math_by_grade_df['12th'].map("{:.2f}".format)

# Print Data

math_by_grade_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.08</td>
      <td>77.00</td>
      <td>77.52</td>
      <td>76.49</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.09</td>
      <td>83.15</td>
      <td>82.77</td>
      <td>83.28</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.40</td>
      <td>76.54</td>
      <td>76.88</td>
      <td>77.15</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.36</td>
      <td>77.67</td>
      <td>76.92</td>
      <td>76.18</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.04</td>
      <td>84.23</td>
      <td>83.84</td>
      <td>83.36</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.44</td>
      <td>77.34</td>
      <td>77.14</td>
      <td>77.19</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.79</td>
      <td>83.43</td>
      <td>85.00</td>
      <td>82.86</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.03</td>
      <td>75.91</td>
      <td>76.45</td>
      <td>77.23</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.19</td>
      <td>76.69</td>
      <td>77.49</td>
      <td>76.86</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.63</td>
      <td>83.37</td>
      <td>84.33</td>
      <td>84.12</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.86</td>
      <td>76.61</td>
      <td>76.40</td>
      <td>77.69</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.42</td>
      <td>82.92</td>
      <td>83.38</td>
      <td>83.78</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.59</td>
      <td>83.09</td>
      <td>83.50</td>
      <td>83.50</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.09</td>
      <td>83.72</td>
      <td>83.20</td>
      <td>83.04</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.26</td>
      <td>84.01</td>
      <td>83.84</td>
      <td>83.64</td>
    </tr>
  </tbody>
</table>

### Reading Scores by Grade


```python
# Calculate Average Reading Scores by grade for each school

reading_by_grade_df = grade_9_df.groupby(['school']).agg({'reading_score':['mean']})
reading_by_grade_df['10th'] = grade_10_df.groupby(['school']).agg({'reading_score':['mean']})
reading_by_grade_df['11th'] = grade_11_df.groupby(['school']).agg({'reading_score':['mean']})
reading_by_grade_df['12th'] = grade_12_df.groupby(['school']).agg({'reading_score':['mean']})

# Format dataframe

reading_by_grade_df.columns = ['9th', '10th', '11th', '12th']

# Format data

reading_by_grade_df['9th'] = reading_by_grade_df['9th'].map("{:.2f}".format)
reading_by_grade_df['10th'] = reading_by_grade_df['10th'].map("{:.2f}".format)
reading_by_grade_df['11th'] = reading_by_grade_df['11th'].map("{:.2f}".format)
reading_by_grade_df['12th'] = reading_by_grade_df['12th'].map("{:.2f}".format)

# Print Data

reading_by_grade_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.30</td>
      <td>80.91</td>
      <td>80.95</td>
      <td>80.91</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.68</td>
      <td>84.25</td>
      <td>83.79</td>
      <td>84.29</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.20</td>
      <td>81.41</td>
      <td>80.64</td>
      <td>81.38</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.63</td>
      <td>81.26</td>
      <td>80.40</td>
      <td>80.66</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.37</td>
      <td>83.71</td>
      <td>84.29</td>
      <td>84.01</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.87</td>
      <td>80.66</td>
      <td>81.40</td>
      <td>80.86</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.68</td>
      <td>83.32</td>
      <td>83.82</td>
      <td>84.70</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.29</td>
      <td>81.51</td>
      <td>81.42</td>
      <td>80.31</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.26</td>
      <td>80.77</td>
      <td>80.62</td>
      <td>81.23</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.81</td>
      <td>83.61</td>
      <td>84.34</td>
      <td>84.59</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.99</td>
      <td>80.63</td>
      <td>80.86</td>
      <td>80.38</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.12</td>
      <td>83.44</td>
      <td>84.37</td>
      <td>82.78</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.73</td>
      <td>84.25</td>
      <td>83.59</td>
      <td>83.83</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.94</td>
      <td>84.02</td>
      <td>83.76</td>
      <td>84.32</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.83</td>
      <td>83.81</td>
      <td>84.16</td>
      <td>84.07</td>
    </tr>
  </tbody>
</table>

### Scores by School Spending


```python
# Create and assign Per Student Budget bins

school_spending_df = df
bins = [570, 599, 629, 649, 659]
group_names = ['< $600', '$600-629', '$630-649', '$650-659']
school_spending_df["Spending Ranges (Per Student)"] = pd.cut(school_spending_df["Per Student Spending"], bins, labels=group_names)

# Group and aggregate data

school_spending_df_grouped = school_spending_df.groupby('Spending Ranges (Per Student)').agg({'reading_score':['mean'], 'math_score':['mean']})
school_spending_summary_df = school_spending_df_grouped.reset_index()
school_spending_summary_df.columns = ['Spending Ranges (Per Student)', 'Average Reading Score', 'Average Math Score']

# Calculate Percent Passing Reading

passing_reading_by_budget = school_spending_df.loc[school_spending_df['reading_score'] >= min_passing_grade, :]
df_passing_reading_by_budget = passing_reading_by_budget.groupby('Spending Ranges (Per Student)')
percent_passing_reading_by_budget = df_passing_reading_by_budget['reading_score'].count() / school_spending_df.groupby('Spending Ranges (Per Student)')['reading_score'].count()
percent_passing_reading_by_budget_df = percent_passing_reading_by_budget.reset_index()
percent_passing_reading_by_budget_df.columns = ['Spending Ranges (Per Student)', '% Passing Reading']

# Calculate Percent Passing Math

passing_math_by_budget = school_spending_df.loc[school_spending_df['math_score'] >= min_passing_grade, :]
df_passing_math_by_budget = passing_math_by_budget.groupby('Spending Ranges (Per Student)')
percent_passing_math_by_budget = df_passing_math_by_budget['math_score'].count() / school_spending_df.groupby('Spending Ranges (Per Student)')['math_score'].count()
percent_passing_math_by_budget_df = percent_passing_math_by_budget.reset_index()
percent_passing_math_by_budget_df.columns = ['Spending Ranges (Per Student)', '% Passing Math']

# Combine dataframes and calculate Overall Passing Rate

percent_passing_by_budget_df = pd.merge(percent_passing_reading_by_budget_df, percent_passing_math_by_budget_df, on="Spending Ranges (Per Student)")
merged_spending_summary_df = pd.merge(school_spending_summary_df, percent_passing_by_budget_df, on="Spending Ranges (Per Student)")
merged_spending_summary_df['% Overall Passing Rate'] = (merged_spending_summary_df['% Passing Reading'] + merged_spending_summary_df['% Passing Math']) / 2

# Format dataframe

merged_spending_summary_df = merged_spending_summary_df.set_index('Spending Ranges (Per Student)')
merged_spending_summary_df = merged_spending_summary_df[['Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]

# Format data

merged_spending_summary_df['Average Math Score'] = merged_spending_summary_df['Average Math Score'].map("{:.2f}".format)
merged_spending_summary_df['Average Reading Score'] = merged_spending_summary_df['Average Reading Score'].map("{:.2f}".format)
merged_spending_summary_df['% Passing Math'] = (merged_spending_summary_df['% Passing Math'] * 100).map("{:.2f}%".format)
merged_spending_summary_df['% Passing Reading'] = (merged_spending_summary_df['% Passing Reading'] * 100).map("{:.2f}%".format)
merged_spending_summary_df['% Overall Passing Rate'] = (merged_spending_summary_df['% Overall Passing Rate'] * 100).map("{:.2f}%".format)

# Print Data

merged_spending_summary_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt; $600</th>
      <td>83.36</td>
      <td>83.96</td>
      <td>93.70%</td>
      <td>96.69%</td>
      <td>95.19%</td>
    </tr>
    <tr>
      <th>$600-629</th>
      <td>79.98</td>
      <td>82.31</td>
      <td>79.11%</td>
      <td>88.51%</td>
      <td>83.81%</td>
    </tr>
    <tr>
      <th>$630-649</th>
      <td>77.82</td>
      <td>81.30</td>
      <td>70.62%</td>
      <td>82.60%</td>
      <td>76.61%</td>
    </tr>
    <tr>
      <th>$650-659</th>
      <td>77.05</td>
      <td>81.01</td>
      <td>66.23%</td>
      <td>81.11%</td>
      <td>73.67%</td>
    </tr>
  </tbody>
</table>

### Scores by School Size


```python
# Create and assign school size bins

school_size_df = df
bins = [0, 1999, 2999, 4999]
group_names = ['Small (< 2000)', 'Medium (2000-2999)', 'Large (3000+)']
school_size_df["School Size"] = pd.cut(school_size_df["size"], bins, labels=group_names)

# Group and aggregate data

school_size_df_grouped = school_size_df.groupby('School Size').agg({'reading_score':['mean'], 'math_score':['mean']})
school_size_summary_df = school_size_df_grouped.reset_index()
school_size_summary_df.columns = ['School Size', 'Average Reading Score', 'Average Math Score']

# Calculate Percent Passing Reading

passing_reading_by_size = school_size_df.loc[school_size_df['reading_score'] >= min_passing_grade, :]
df_passing_reading_by_size = passing_reading_by_size.groupby('School Size')
percent_passing_reading_by_size = df_passing_reading_by_size['reading_score'].count() / school_size_df.groupby('School Size')['reading_score'].count()
percent_passing_reading_by_size_df = percent_passing_reading_by_size.reset_index()
percent_passing_reading_by_size_df.columns = ['School Size', '% Passing Reading']

# Calculate Percent Passing Math

passing_math_by_size = school_size_df.loc[school_size_df['math_score'] >= min_passing_grade, :]
df_passing_math_by_size = passing_math_by_size.groupby('School Size')
percent_passing_math_by_size = df_passing_math_by_size['math_score'].count() / school_size_df.groupby('School Size')['math_score'].count()
percent_passing_math_by_size_df = percent_passing_math_by_size.reset_index()
percent_passing_math_by_size_df.columns = ['School Size', '% Passing Math']

# Combine dataframes and calculate Overall Passing Rate

percent_passing_by_size_df = pd.merge(percent_passing_reading_by_size_df, percent_passing_math_by_size_df, on="School Size")
merged_size_summary_df = pd.merge(school_size_summary_df, percent_passing_by_size_df, on="School Size")
merged_size_summary_df['% Overall Passing Rate'] = (merged_size_summary_df['% Passing Reading'] + merged_size_summary_df['% Passing Math']) / 2

# Format dataframe

merged_size_summary_df = merged_size_summary_df.set_index('School Size')
merged_size_summary_df = merged_size_summary_df[['Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]

# Format data

merged_size_summary_df['Average Math Score'] = merged_size_summary_df['Average Math Score'].map("{:.2f}".format)
merged_size_summary_df['Average Reading Score'] = merged_size_summary_df['Average Reading Score'].map("{:.2f}".format)
merged_size_summary_df['% Passing Math'] = (merged_size_summary_df['% Passing Math'] * 100).map("{:.2f}%".format)
merged_size_summary_df['% Passing Reading'] = (merged_size_summary_df['% Passing Reading'] * 100).map("{:.2f}%".format)
merged_size_summary_df['% Overall Passing Rate'] = (merged_size_summary_df['% Overall Passing Rate'] * 100).map("{:.2f}%".format)

# Print data

merged_size_summary_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt; 2000)</th>
      <td>83.44</td>
      <td>83.88</td>
      <td>93.66%</td>
      <td>96.67%</td>
      <td>95.17%</td>
    </tr>
    <tr>
      <th>Medium (2000-2999)</th>
      <td>78.16</td>
      <td>81.65</td>
      <td>72.34%</td>
      <td>83.84%</td>
      <td>78.09%</td>
    </tr>
    <tr>
      <th>Large (3000+)</th>
      <td>77.07</td>
      <td>80.93</td>
      <td>66.47%</td>
      <td>81.11%</td>
      <td>73.79%</td>
    </tr>
  </tbody>
</table>

### Scores by School Type


```python
# Group and aggregate data by School Type

type_summary = df.groupby('type').agg({'reading_score':['mean'], 'math_score':['mean']})
type_summary_df = type_summary.reset_index()
type_summary_df.columns = ['School Type', 'Average Reading Score', 'Average Math Score']

# Calculate Percent Passing Reading

df_passing_reading = df.loc[df['reading_score'] >= min_passing_grade, :]
df_passing_reading_by_type = df_passing_reading.groupby('type')
percent_passing_reading_by_type = df_passing_reading_by_type['reading_score'].count() / df.groupby('type')['reading_score'].count()
percent_passing_reading_by_type_df = percent_passing_reading_by_type.reset_index()
percent_passing_reading_by_type_df.columns = ['School Type', '% Passing Reading']

# Calculate Percent Passing Math

df_passing_math = df.loc[df['math_score'] >= min_passing_grade, :]
df_passing_math_by_type = df_passing_math.groupby('type')
percent_passing_math_by_type = df_passing_math_by_type['math_score'].count() / df.groupby('type')['math_score'].count()
percent_passing_math_by_type_df = percent_passing_math_by_type.reset_index()
percent_passing_math_by_type_df.columns = ['School Type', '% Passing Math']

# Combine dataframes and calculate Overall Passing Rate

percent_passing_by_type_df = pd.merge(percent_passing_reading_by_type_df, percent_passing_math_by_type_df, on="School Type")
merged_type_summary_df = pd.merge(type_summary_df, percent_passing_by_type_df, on="School Type")
merged_type_summary_df['% Overall Passing Rate'] = (merged_type_summary_df['% Passing Reading'] + merged_type_summary_df['% Passing Math']) / 2

# Format dataframe

merged_type_summary_df = merged_type_summary_df.set_index('School Type')
merged_type_summary_df = merged_type_summary_df[['Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]

# Format data

merged_type_summary_df['Average Math Score'] = merged_type_summary_df['Average Math Score'].map("{:.2f}".format)
merged_type_summary_df['Average Reading Score'] = merged_type_summary_df['Average Reading Score'].map("{:.2f}".format)
merged_type_summary_df['% Passing Math'] = (merged_type_summary_df['% Passing Math'] * 100).map("{:.2f}%".format)
merged_type_summary_df['% Passing Reading'] = (merged_type_summary_df['% Passing Reading'] * 100).map("{:.2f}%".format)
merged_type_summary_df['% Overall Passing Rate'] = (merged_type_summary_df['% Overall Passing Rate'] * 100).map("{:.2f}%".format)

# Print data

merged_type_summary_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.41</td>
      <td>83.90</td>
      <td>93.70%</td>
      <td>96.65%</td>
      <td>95.17%</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.99</td>
      <td>80.96</td>
      <td>66.52%</td>
      <td>80.91%</td>
      <td>73.71%</td>
    </tr>
  </tbody>
</table>
