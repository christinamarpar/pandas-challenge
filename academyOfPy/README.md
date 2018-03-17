

```python
# Import Dependencies
import pandas as pd
import numpy as np
```


```python
# Read in the schools and students files
schools_path = "schools_complete.csv"
schools_df = pd.read_csv(schools_path)

students_path = "students_complete.csv"
students_df = pd.read_csv(students_path)
```


```python
#schools_df.head()
```


```python
#save school df as a pd - I will need this later, for a merge
school_orig_pd = pd.DataFrame(schools_df)

#reset the index to school name
school_orig_pd.set_index(['name'], inplace=True, drop=True)
```


```python
#school_orig_pd.head()
```


```python
#students_df.head()
```


```python
#modify students_df to include binary vars for passing math and reading
students_df['pass_reading'] = np.where(students_df['reading_score']>59, 1, 0)
students_df['pass_math'] = np.where(students_df['math_score']>59, 1, 0)
#students_df.head()
```


```python
#save student df as a pd - I will need this later, for a merge
stud_orig_pd = pd.DataFrame(students_df)

#reset the index to school name
stud_orig_pd.set_index(['school'], inplace=True, drop=True)
```


```python
#stud_orig_pd.head()
```


```python
#Create summary data

#Total Schools
school_count = schools_df['School ID'].value_counts()
nschools = school_count.count()
#Total Students
student_count = students_df['Student ID'].value_counts()
nstudents = student_count.count()
#Total Budget
total_budget = schools_df['budget'].sum()
#Average Math Score
avg_math = round(students_df["math_score"].mean(),0)
#Average Reading Score
avg_reading = round(students_df["reading_score"].mean(),0)
#% Passing Math
n_passing_math = sum(i > 59 for i in students_df['math_score'])
p_passing_math=100*round(n_passing_math/nstudents,4)
#% Passing Reading
n_passing_reading = sum(i > 59 for i in students_df['reading_score'])
p_passing_reading=100*round(n_passing_reading/nstudents,4)
#Overall Passing Rate (Average of the above two)
p_passing_overall = round((p_passing_math+p_passing_reading)/2,2)
```


```python
#Create the summary table
sumstats_df = pd.DataFrame(
    {"Total Schools": [nschools],
     "Total Students": [nstudents],
     "Total Budget": [total_budget],
     "Average Math Score": [avg_math],
     "Average Reading Score": [avg_reading],
     "% Passing Math": [p_passing_math],
     "% Passing Reading": [p_passing_reading],
     "% Passing Overall": [p_passing_overall]
    })

#Fill in summary table
sumstats_df["Total Students"] = sumstats_df["Total Students"].map("{:,.0f}".format)
sumstats_df["Total Budget"] = sumstats_df["Total Budget"].map("${:,.0f}".format)
sumstats_df["Average Math Score"] = sumstats_df["Average Math Score"].map("{:,.0f}".format)
sumstats_df["Average Reading Score"] = sumstats_df["Average Reading Score"].map("{:,.0f}".format)

sumstats_clean_df = sumstats_df[['Total Schools','Total Students','Total Budget',
                                 'Average Math Score','Average Reading Score','% Passing Math','% Passing Reading',
                                 '% Passing Overall']]
sumstats_clean_df
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
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428</td>
      <td>79</td>
      <td>82</td>
      <td>92.45</td>
      <td>100.0</td>
      <td>96.22</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Group by school
group_by_school = students_df.groupby(['school'])
```


```python
#Save means
school_means_pd = pd.DataFrame(
    group_by_school['math_score','reading_score','pass_reading','pass_math'].mean()
)
school_means_pd['pass_math']=round(school_means_pd['pass_math']*100,2)
school_means_pd['pass_reading']=round(school_means_pd['pass_reading']*100,2)
school_means_pd['pass_both']=round(
    (school_means_pd['pass_reading']+school_means_pd['pass_math'])/2,2)

#school_means_pd.head()
```


```python
#rename columns of means
school_means_pd_clean = school_means_pd.rename(columns={"math_score": "Average Math Score",
                                                    "reading_score": "Average Reading Score",
                                                    "pass_reading": "% Passing Reading",
                                                    "pass_math": "% Passing Math",
                                                    "pass_both": "% Passing Overall"
                                               })
#school_means_pd_clean.head()
```


```python
#merge the means (m) with the original (o) school df
school_om = school_orig_pd.join(school_means_pd_clean, how='outer')
#school_om.head()
```


```python
#rename original school df columns
school_om_clean = school_om.rename(columns={"type": "School Type",
                                            "size": "Total Students",
                                            "budget": "Total School Budget"
                                               })
#school_om_clean.head()
```


```python
#Add in the per student budget column
school_om_clean['Per Student Budget'] = school_om_clean['Total School Budget']/school_om_clean['Total Students']
#school_om_clean.head()
```


```python
#Drop School ID, reorder vars
school_om_red = school_om_clean[['School Type','Total Students','Total School Budget',
                               'Per Student Budget', 'Average Math Score', 
                                'Average Reading Score', '% Passing Math',
                               '% Passing Reading', '% Passing Overall']]
#school_om_red.head()
```


```python
#Format data
school_summary = school_om_red[:]
school_summary["Total Students"] = school_summary["Total Students"].map("{:,.0f}".format)
school_summary["Total School Budget"] = school_summary["Total School Budget"].map("${:,.0f}".format)
school_summary["Per Student Budget"] = school_summary["Per Student Budget"].map("${:,.0f}".format)
school_summary["Average Math Score"] = school_summary["Average Math Score"].map("{:,.0f}".format)
school_summary["Average Reading Score"] = school_summary["Average Reading Score"].map("{:,.0f}".format)

school_summary["% Passing Reading"] = school_summary["% Passing Reading"].map("{:,.2f}".format)
school_summary["% Passing Math"] = school_summary["% Passing Math"].map("{:,.2f}".format)
school_summary["% Passing Overall"] = school_summary["% Passing Overall"].map("{:,.2f}".format)

school_summary.head()
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
      <th>wtdavg_math</th>
      <th>wtdavg_reading</th>
      <th>Budget Group</th>
      <th>wtdavg_pct_math</th>
      <th>wtdavg_pct_reading</th>
      <th>School Size</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4,976</td>
      <td>$3,124,928</td>
      <td>$628</td>
      <td>77</td>
      <td>81</td>
      <td>89.53</td>
      <td>100.00</td>
      <td>94.76</td>
      <td>383393.0</td>
      <td>403225.0</td>
      <td>[$625,$650)</td>
      <td>445501.28</td>
      <td>497600.0</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>$582</td>
      <td>83</td>
      <td>84</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>154329.0</td>
      <td>156027.0</td>
      <td>[$575,$600)</td>
      <td>185800.00</td>
      <td>185800.0</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>$639</td>
      <td>77</td>
      <td>81</td>
      <td>88.44</td>
      <td>100.00</td>
      <td>94.22</td>
      <td>226223.0</td>
      <td>239335.0</td>
      <td>[$625,$650)</td>
      <td>260809.56</td>
      <td>294900.0</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916</td>
      <td>$644</td>
      <td>77</td>
      <td>81</td>
      <td>89.30</td>
      <td>100.00</td>
      <td>94.65</td>
      <td>211184.0</td>
      <td>221164.0</td>
      <td>[$625,$650)</td>
      <td>244592.70</td>
      <td>273900.0</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>$625</td>
      <td>83</td>
      <td>84</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>122360.0</td>
      <td>123043.0</td>
      <td>[$600,$625)</td>
      <td>146800.00</td>
      <td>146800.0</td>
      <td>Small</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Check to see that this was not POINTED to
#school_om_red.head()
```


```python
#Sort descending
top_schools = school_om_red[:]
top_schools = top_schools.sort_values(["% Passing Overall"], ascending=[False])

#Format
top_schools["Total Students"] = top_schools["Total Students"].map("{:,.0f}".format)
top_schools["Total School Budget"] = top_schools["Total School Budget"].map("${:,.0f}".format)
top_schools["Per Student Budget"] = top_schools["Per Student Budget"].map("${:,.0f}".format)
top_schools["Average Math Score"] = top_schools["Average Math Score"].map("{:,.0f}".format)
top_schools["Average Reading Score"] = top_schools["Average Reading Score"].map("{:,.0f}".format)

top_schools["% Passing Reading"] = top_schools["% Passing Reading"].map("{:,.2f}".format)
top_schools["% Passing Math"] = top_schools["% Passing Math"].map("{:,.2f}".format)
top_schools["% Passing Overall"] = top_schools["% Passing Overall"].map("{:,.2f}".format)

top_schools.head()
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>$582</td>
      <td>83</td>
      <td>84</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>100.00</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>$625</td>
      <td>83</td>
      <td>84</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>100.00</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087</td>
      <td>$581</td>
      <td>84</td>
      <td>84</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>100.00</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609</td>
      <td>84</td>
      <td>84</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>100.00</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1,761</td>
      <td>$1,056,600</td>
      <td>$600</td>
      <td>83</td>
      <td>84</td>
      <td>100.00</td>
      <td>100.00</td>
      <td>100.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Sort ascending
worst_schools = school_om_red[:]
worst_schools = worst_schools.sort_values(["% Passing Overall"], ascending=[True])

#Format
worst_schools["Total Students"] = worst_schools["Total Students"].map("{:,.0f}".format)
worst_schools["Total School Budget"] = worst_schools["Total School Budget"].map("${:,.0f}".format)
worst_schools["Per Student Budget"] = worst_schools["Per Student Budget"].map("${:,.0f}".format)
worst_schools["Average Math Score"] = worst_schools["Average Math Score"].map("{:,.0f}".format)
worst_schools["Average Reading Score"] = worst_schools["Average Reading Score"].map("{:,.0f}".format)

worst_schools["% Passing Reading"] = worst_schools["% Passing Reading"].map("{:,.2f}".format)
worst_schools["% Passing Math"] = worst_schools["% Passing Math"].map("{:,.2f}".format)
worst_schools["% Passing Overall"] = worst_schools["% Passing Overall"].map("{:,.2f}".format)

worst_schools.head()
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
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>$639</td>
      <td>77</td>
      <td>81</td>
      <td>88.44</td>
      <td>100.00</td>
      <td>94.22</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363</td>
      <td>$637</td>
      <td>77</td>
      <td>81</td>
      <td>88.55</td>
      <td>100.00</td>
      <td>94.28</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635</td>
      <td>$655</td>
      <td>77</td>
      <td>81</td>
      <td>88.86</td>
      <td>100.00</td>
      <td>94.43</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020</td>
      <td>$652</td>
      <td>77</td>
      <td>81</td>
      <td>89.08</td>
      <td>100.00</td>
      <td>94.54</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650</td>
      <td>$650</td>
      <td>77</td>
      <td>81</td>
      <td>89.18</td>
      <td>100.00</td>
      <td>94.59</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Group by school and grade
group_by_school_grade = students_df.groupby(['school','grade'])
```


```python
#Mean math score
schools_grade_math_pd = pd.DataFrame(
    group_by_school_grade['math_score'].mean()
)
schools_grade_math_pd.head()
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
      <th></th>
      <th>math_score</th>
    </tr>
    <tr>
      <th>school</th>
      <th>grade</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">Bailey High School</th>
      <th>10th</th>
      <td>76.996772</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>77.515588</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>9th</th>
      <td>77.083676</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <th>10th</th>
      <td>83.154506</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Mean reading score
schools_grade_reading_pd = pd.DataFrame(
    group_by_school_grade['reading_score'].mean()
)
schools_grade_reading_pd.head()
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
      <th></th>
      <th>reading_score</th>
    </tr>
    <tr>
      <th>school</th>
      <th>grade</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">Bailey High School</th>
      <th>10th</th>
      <td>80.907183</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>80.945643</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>9th</th>
      <td>81.303155</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <th>10th</th>
      <td>84.253219</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Create weighted variables prior to the groupby
school_om_red["wtdavg_math"]=(school_om_red["Total Students"]*school_om_red["Average Math Score"])
school_om_red["wtdavg_reading"]=(school_om_red["Total Students"]*school_om_red["Average Reading Score"])
school_om_red["wtdavg_pct_math"]=(school_om_red["Total Students"]*school_om_red["% Passing Math"])
school_om_red["wtdavg_pct_reading"]=(school_om_red["Total Students"]*school_om_red["% Passing Reading"])
```


```python
#Inform psbudget bin cutoffs
#school_om_red["Per Student Budget"].describe()
```


```python
# Create the per student school budget bins
bins = [575, 600, 625, 650, 675]
# Create the names for the four bins
group_names = ['[$575,$600)', '[$600,$625)', '[$625,$650)', '[$650,$675)']
```


```python
#Create a 'Budget Group' column using bins
school_om_red["Budget Group"] = pd.cut(school_om_red["Per Student Budget"], bins, labels=group_names)
```


```python
#school_om_red.head()
```


```python
#Groupby budget group
group_by_psbudget = school_om_red.groupby(['Budget Group'])

bypsbudget = pd.DataFrame(
    group_by_psbudget['Total Students','wtdavg_math','wtdavg_reading','wtdavg_pct_math','wtdavg_pct_reading'].sum()
)
```


```python
#Create desired vars
bypsbudget["Average Math Score"] = bypsbudget["wtdavg_math"]/bypsbudget["Total Students"]
bypsbudget["Average Reading Score"] = bypsbudget["wtdavg_reading"]/bypsbudget["Total Students"]
bypsbudget["% Passing Math"] = bypsbudget["wtdavg_pct_math"]/bypsbudget["Total Students"]
bypsbudget["% Passing Reading"] = bypsbudget["wtdavg_pct_reading"]/bypsbudget["Total Students"]
bypsbudget["% Passing Overall"] = (bypsbudget["% Passing Math"]+bypsbudget["% Passing Reading"])/2 
```


```python
#bypsbudget.head()
```


```python
#Reduce the number of columns
bypsbudget_red = bypsbudget[['Average Math Score','Average Reading Score',
                               '% Passing Math', '% Passing Reading', 
                                '% Passing Overall']]
```


```python
bypsbudget_red.head()
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Budget Group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>[$575,$600)</th>
      <td>83.362283</td>
      <td>83.912412</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>[$600,$625)</th>
      <td>83.544856</td>
      <td>83.906996</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>[$625,$650)</th>
      <td>77.469253</td>
      <td>81.162258</td>
      <td>89.895103</td>
      <td>100.0</td>
      <td>94.947551</td>
    </tr>
    <tr>
      <th>[$650,$675)</th>
      <td>77.034693</td>
      <td>81.030323</td>
      <td>88.995024</td>
      <td>100.0</td>
      <td>94.497512</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Inform school size bin cutoffs
#school_om_red['Total Students'].describe()
```


```python
# Create the school size bins
bins = [350, 1900, 3450, 5000]
# Create the names for the bins
group_names = ['Small', 'Medium', 'Large']
```


```python
#Create a 'School Size' column using bins
school_om_red["School Size"] = pd.cut(school_om_red["Total Students"], bins, labels=group_names)
```


```python
#school_om_red.head()
```


```python
#Groupby school size
group_by_size = school_om_red.groupby(['School Size'])

bysize = pd.DataFrame(
    group_by_size['Total Students','wtdavg_math','wtdavg_reading','wtdavg_pct_math','wtdavg_pct_reading'].sum()
)
```


```python
#Create desired vars
bysize["Average Math Score"] = bysize["wtdavg_math"]/bysize["Total Students"]
bysize["Average Reading Score"] = bysize["wtdavg_reading"]/bysize["Total Students"]
bysize["% Passing Math"] = bysize["wtdavg_pct_math"]/bysize["Total Students"]
bysize["% Passing Reading"] = bysize["wtdavg_pct_reading"]/bysize["Total Students"]
bysize["% Passing Overall"] = (bysize["% Passing Math"]+bysize["% Passing Reading"])/2
```


```python
#Reduce the number of columns
bysize_red = bysize[['Average Math Score','Average Reading Score',
                               '% Passing Math', '% Passing Reading', 
                                '% Passing Overall']]
```


```python
bysize_red.head()
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
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
      <th>Small</th>
      <td>83.436586</td>
      <td>83.882857</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>Medium</th>
      <td>78.164034</td>
      <td>81.654758</td>
      <td>91.192770</td>
      <td>100.0</td>
      <td>95.596385</td>
    </tr>
    <tr>
      <th>Large</th>
      <td>77.070764</td>
      <td>80.928365</td>
      <td>89.112433</td>
      <td>100.0</td>
      <td>94.556217</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Groupby school type
group_by_type = school_om_red.groupby(['School Type'])

bytype = pd.DataFrame(
    group_by_type['Total Students','wtdavg_math','wtdavg_reading','wtdavg_pct_math','wtdavg_pct_reading'].sum()
)
```


```python
#Create desired vars
bytype["Average Math Score"] = bytype["wtdavg_math"]/bytype["Total Students"]
bytype["Average Reading Score"] = bytype["wtdavg_reading"]/bytype["Total Students"]
bytype["% Passing Math"] = bytype["wtdavg_pct_math"]/bytype["Total Students"]
bytype["% Passing Reading"] = bytype["wtdavg_pct_reading"]/bytype["Total Students"]
bytype["% Passing Overall"] = (bytype["% Passing Math"]+bytype["% Passing Reading"])/2

```


```python
#Reduce the number of columns
bytype_red = bytype[['Average Math Score','Average Reading Score',
                               '% Passing Math', '% Passing Reading', 
                                '% Passing Overall']]
```


```python
bytype_red.head()
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
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
      <td>83.406183</td>
      <td>83.902821</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.987026</td>
      <td>80.962485</td>
      <td>89.030671</td>
      <td>100.0</td>
      <td>94.515336</td>
    </tr>
  </tbody>
</table>
</div>


