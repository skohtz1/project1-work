

```python
import os
import csv
import pandas as pd
import numpy as np
import warnings
warnings.filterwarnings('ignore')
from scipy import stats
import matplotlib.pyplot as plt

# ##The average commute time for transit users.
# Percentage difference between average commute times of car commuters and transit users.
# Percentage of commuters who use public transit.
# Total number of commuters who use public transit.
# The difference between the citywide median income and the median income of transit users.

```


```python
##All csvs

commutercsv = "commutertimes1.csv"
commuterpubliccsv = "commutertimespublic.csv"
populationcsv = "population1.csv"
citiescsv = "cities1.csv"
commuting_driving = "commutingdriving.csv"
```


```python
##Reading the csvs, changing some names of columns to match certain csvs, making the cities lower case, the "thousands = ,"makes sure that the string 1,200 is a number

commuter_df = pd.read_csv(commutercsv, thousands = ",")
commuter_public_df = pd.read_csv(commuterpubliccsv, thousands = ",")
population_df = pd.read_csv(populationcsv, thousands = ",")
cities_df = pd.read_csv(citiescsv)
commuting_driving_df = pd.read_csv(commuting_driving, thousands = ",")
new_pop_df = population_df[["NAME","STNAME","POPESTIMATE2016"]]
cities_df=cities_df.rename(columns={"City/Region": "City"})
commuter_public_df["City"]=commuter_public_df["City"].str.lower()
commuting_driving_df["City"]= commuting_driving_df["City"].str.lower()
cities_df["City"]=cities_df["City"].str.lower()
commuter_df["City"]=commuter_df["City"].str.lower()
new_pop_df["NAME"]=new_pop_df["NAME"].str.lower()
commuting_driving_df = commuting_driving_df.dropna(axis=0,how="any")


```


```python
##This is the dataset of all cities and average commute time for driving in minutes
commuter_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>Country</th>
      <th>GeoID</th>
      <th>City</th>
      <th>State</th>
      <th>Area Type</th>
      <th>Minutes</th>
      <th>Margin of Error</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0100000US</td>
      <td>United States</td>
      <td>310M300US10100</td>
      <td>aberdeen</td>
      <td>SD</td>
      <td>Micro</td>
      <td>13.2</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0100000US</td>
      <td>United States</td>
      <td>310M300US10140</td>
      <td>aberdeen</td>
      <td>WA</td>
      <td>Micro</td>
      <td>24.8</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0100000US</td>
      <td>United States</td>
      <td>310M300US10180</td>
      <td>abilene</td>
      <td>TX</td>
      <td>Metro</td>
      <td>17.4</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0100000US</td>
      <td>United States</td>
      <td>310M300US10220</td>
      <td>ada</td>
      <td>OK</td>
      <td>Micro</td>
      <td>17.3</td>
      <td>0.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0100000US</td>
      <td>United States</td>
      <td>310M300US10300</td>
      <td>adrian</td>
      <td>MI</td>
      <td>Micro</td>
      <td>26.5</td>
      <td>0.7</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Populatino estimates for each city and state
new_pop_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NAME</th>
      <th>STNAME</th>
      <th>POPESTIMATE2016</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>alabama</td>
      <td>Alabama</td>
      <td>4863300</td>
    </tr>
    <tr>
      <th>1</th>
      <td>abbeville city</td>
      <td>Alabama</td>
      <td>2603</td>
    </tr>
    <tr>
      <th>2</th>
      <td>adamsville city</td>
      <td>Alabama</td>
      <td>4360</td>
    </tr>
    <tr>
      <th>3</th>
      <td>addison town</td>
      <td>Alabama</td>
      <td>738</td>
    </tr>
    <tr>
      <th>4</th>
      <td>akron town</td>
      <td>Alabama</td>
      <td>334</td>
    </tr>
  </tbody>
</table>
</div>




```python
##List of Cities of interest
cities_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>atlanta</td>
      <td>GA</td>
    </tr>
    <tr>
      <th>1</th>
      <td>austin</td>
      <td>TX</td>
    </tr>
    <tr>
      <th>2</th>
      <td>boston</td>
      <td>MA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>chicago</td>
      <td>IL</td>
    </tr>
    <tr>
      <th>4</th>
      <td>columbus</td>
      <td>OH</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Public transportation data (needs cleaning)
commuter_public_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>City</th>
      <th>State</th>
      <th>Legacy NTD ID</th>
      <th>NTD ID</th>
      <th>Organization Type</th>
      <th>NTD Reporter Type</th>
      <th>Primary UZA Population</th>
      <th>Agency VOMS</th>
      <th>Mode</th>
      <th>...</th>
      <th>Unnamed: 37</th>
      <th>Unnamed: 38</th>
      <th>Unnamed: 39</th>
      <th>Unnamed: 40</th>
      <th>Unnamed: 41</th>
      <th>Unnamed: 42</th>
      <th>Unnamed: 43</th>
      <th>Unnamed: 44</th>
      <th>Unnamed: 45</th>
      <th>Unnamed: 46</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MTA New York City Transit</td>
      <td>new york</td>
      <td>NY</td>
      <td>2008</td>
      <td>20008</td>
      <td>Subsidiary Unit of a Transit Agency, Reporting...</td>
      <td>Full Reporter</td>
      <td>18351295</td>
      <td>11004</td>
      <td>HR</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MTA New York City Transit</td>
      <td>new york</td>
      <td>NY</td>
      <td>2008</td>
      <td>20008</td>
      <td>Subsidiary Unit of a Transit Agency, Reporting...</td>
      <td>Full Reporter</td>
      <td>18351295</td>
      <td>11004</td>
      <td>MB</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MTA New York City Transit</td>
      <td>new york</td>
      <td>NY</td>
      <td>2008</td>
      <td>20008</td>
      <td>Subsidiary Unit of a Transit Agency, Reporting...</td>
      <td>Full Reporter</td>
      <td>18351295</td>
      <td>11004</td>
      <td>DR</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MTA New York City Transit</td>
      <td>new york</td>
      <td>NY</td>
      <td>2008</td>
      <td>20008</td>
      <td>Subsidiary Unit of a Transit Agency, Reporting...</td>
      <td>Full Reporter</td>
      <td>18351295</td>
      <td>11004</td>
      <td>CB</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MTA New York City Transit</td>
      <td>new york</td>
      <td>NY</td>
      <td>2008</td>
      <td>20008</td>
      <td>Subsidiary Unit of a Transit Agency, Reporting...</td>
      <td>Full Reporter</td>
      <td>18351295</td>
      <td>11004</td>
      <td>RB</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 47 columns</p>
</div>




```python
##Creating neww columns in the cities_df 
cities_df["Average Public Transport Commute (minutes)"]= ""
cities_df["Average Individual Commute (distance)"] = ""
cities_df["Population"] = ""
cities_df["Population Served"]=""
```


```python
##getting rid of unwanted columns
try:
    for i in range(30,47):
        del commuter_public_df[commuter_public_df.columns[i]]
except IndexError:
    pass
```


```python
##Yes this is the same cell as before..run it
try:
    for i in range(30,47):
        del commuter_public_df[commuter_public_df.columns[i]]
except IndexError:
    pass
```


```python
##Still getting rid of them...
del commuter_public_df["Unnamed: 33"]
del commuter_public_df["Unnamed: 37"]
```


```python
#And...that's better
del commuter_public_df["Unnamed: 41"]
del commuter_public_df["Unnamed: 45"]
```


```python
##Extracting from the csv, the total number of people that use car, truck or public transportation. We only want our cities of interest

list_of_df3 = []
merged_df3 = []
for index,row in cities_df.iterrows():
    ge3 = commuting_driving_df[commuting_driving_df["City"].str.contains(row["City"])]
    ga3 = ge3[ge3["State"].str.contains(row["State"])][["City","State","Mode of Transport","Total People","Margin of Error"]]
 #   print(row["State"])
    list_of_df3.append(ga3)
    
merged_df3 = pd.concat(list_of_df3)
merged_df3.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>State</th>
      <th>Mode of Transport</th>
      <th>Total People</th>
      <th>Margin of Error</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>49</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Drove alone</td>
      <td>34</td>
      <td>55</td>
    </tr>
    <tr>
      <th>50</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Other travel mode</td>
      <td>30</td>
      <td>45</td>
    </tr>
    <tr>
      <th>51</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Drove alone</td>
      <td>79</td>
      <td>63</td>
    </tr>
    <tr>
      <th>237</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Drove alone</td>
      <td>51</td>
      <td>35</td>
    </tr>
    <tr>
      <th>238</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Carpooled</td>
      <td>16</td>
      <td>26</td>
    </tr>
  </tbody>
</table>
</div>




```python
##For this data set, we are only concerned about people that are using their cars. So, we need to get rid of rows with public transport and "other travel mode"

merged_df3_drive_only=merged_df3[merged_df3["Mode of Transport"]!="Public transportation"]
merged_df3_drive_only=merged_df3_drive_only[merged_df3_drive_only["Mode of Transport"]!="Other travel mode"]

merged_df3_drive_only.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>State</th>
      <th>Mode of Transport</th>
      <th>Total People</th>
      <th>Margin of Error</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>49</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Drove alone</td>
      <td>34</td>
      <td>55</td>
    </tr>
    <tr>
      <th>51</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Drove alone</td>
      <td>79</td>
      <td>63</td>
    </tr>
    <tr>
      <th>237</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Drove alone</td>
      <td>51</td>
      <td>35</td>
    </tr>
    <tr>
      <th>238</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Carpooled</td>
      <td>16</td>
      <td>26</td>
    </tr>
    <tr>
      <th>260</th>
      <td>atlanta-sandy springs-roswell</td>
      <td>GA</td>
      <td>Car, truck, or van: Drove alone</td>
      <td>50</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Camden and Rockville are considered interchangeable due to how close they are in MD, and they both reside in Montgomery county
##sums_driving is a dctionary that counts up how many people drive to work per year, and the keys are the cities

sums_driving = {}
cities_df.set_value(10, 'City', "camden")
for index,row in cities_df.iterrows():
    sum_of_people = merged_df3_drive_only[merged_df3_drive_only["City"].str.contains(row["City"])]
    sum_of_driving_people = sum_of_people[sum_of_people["State"].str.contains(row["State"])]
    sums_driving[row["City"]]=sum_of_driving_people["Total People"].sum()

cities_df = cities_df.set_value(10,"City","rockville")
sums_driving
```




    {'alexandria': 4830932,
     'atlanta': 2143375,
     'austin': 774492,
     'boston': 1912936,
     'camden': 2207525,
     'chicago': 3509505,
     'columbus': 857407,
     'dallas': 2850110,
     'denver': 1095293,
     'indianapolis': 828224,
     'los angeles': 5103600,
     'miami': 2235321,
     'nashville': 751647,
     'new york': 10578606,
     'newark': 10578606,
     'philadelphia': 2207525,
     'pittsburgh': 959786,
     'raleigh': 487275,
     'washington': 4830932}




```python
##sum_driving needs to be converted into a dataframe, with an index

sum_driving_df = pd.DataFrame.from_dict(sums_driving, orient = "index")
sum_driving_df = sum_driving_df.rename(columns = {0:"Total People per year"})
```


```python
sum_driving_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total People per year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>atlanta</th>
      <td>2143375</td>
    </tr>
    <tr>
      <th>austin</th>
      <td>774492</td>
    </tr>
    <tr>
      <th>boston</th>
      <td>1912936</td>
    </tr>
    <tr>
      <th>chicago</th>
      <td>3509505</td>
    </tr>
    <tr>
      <th>columbus</th>
      <td>857407</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Making sure the cities are not the index, but an actual index
sum_driving_df.reset_index(level=0, inplace=True)
```


```python
sum_driving_df = sum_driving_df.rename(columns = {"index":"City"})
```


```python
sum_driving_df.set_value(10,"City","rockville")
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Total People per year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>atlanta</td>
      <td>2143375</td>
    </tr>
    <tr>
      <th>1</th>
      <td>austin</td>
      <td>774492</td>
    </tr>
    <tr>
      <th>2</th>
      <td>boston</td>
      <td>1912936</td>
    </tr>
    <tr>
      <th>3</th>
      <td>chicago</td>
      <td>3509505</td>
    </tr>
    <tr>
      <th>4</th>
      <td>columbus</td>
      <td>857407</td>
    </tr>
    <tr>
      <th>5</th>
      <td>dallas</td>
      <td>2850110</td>
    </tr>
    <tr>
      <th>6</th>
      <td>denver</td>
      <td>1095293</td>
    </tr>
    <tr>
      <th>7</th>
      <td>indianapolis</td>
      <td>828224</td>
    </tr>
    <tr>
      <th>8</th>
      <td>los angeles</td>
      <td>5103600</td>
    </tr>
    <tr>
      <th>9</th>
      <td>miami</td>
      <td>2235321</td>
    </tr>
    <tr>
      <th>10</th>
      <td>rockville</td>
      <td>2207525</td>
    </tr>
    <tr>
      <th>11</th>
      <td>nashville</td>
      <td>751647</td>
    </tr>
    <tr>
      <th>12</th>
      <td>newark</td>
      <td>10578606</td>
    </tr>
    <tr>
      <th>13</th>
      <td>new york</td>
      <td>10578606</td>
    </tr>
    <tr>
      <th>14</th>
      <td>alexandria</td>
      <td>4830932</td>
    </tr>
    <tr>
      <th>15</th>
      <td>philadelphia</td>
      <td>2207525</td>
    </tr>
    <tr>
      <th>16</th>
      <td>pittsburgh</td>
      <td>959786</td>
    </tr>
    <tr>
      <th>17</th>
      <td>raleigh</td>
      <td>487275</td>
    </tr>
    <tr>
      <th>18</th>
      <td>washington</td>
      <td>4830932</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Adding another row, the future data sets have "passengers per hour" so this makes it consistent with the other dataframe
##Also, keep in mind this dataset shows the city as the commuting destination not residence
sum_driving_df["Average People per Hour"] = (sum_driving_df["Total People per year"]/365)/24
```


```python
#We no longer need this column
del sum_driving_df["Total People per year"]
```


```python
##Tranporation mode symbols
##Mode Symbol:
##CB: Commuter Bus
##DR: Demand Response
##HR: Heavy Rail
##FB: Ferryboat
##MB: Bus
##CB: Commuter Bus
##DT: Demand Response Taxi (how NY has none of this..)
##CR: Commuter Rail
##VP: Vanpool
##LR: Light Rail
##SR: Streetcar Rail
##TB: Trolleybus
##RB: Bus Rapid Transit
##MG: Monorail/Automated Guideway
##YR: Hybrid Rail
sum_driving_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Average People per Hour</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>atlanta</td>
      <td>244.677511</td>
    </tr>
    <tr>
      <th>1</th>
      <td>austin</td>
      <td>88.412329</td>
    </tr>
    <tr>
      <th>2</th>
      <td>boston</td>
      <td>218.371689</td>
    </tr>
    <tr>
      <th>3</th>
      <td>chicago</td>
      <td>400.628425</td>
    </tr>
    <tr>
      <th>4</th>
      <td>columbus</td>
      <td>97.877511</td>
    </tr>
    <tr>
      <th>5</th>
      <td>dallas</td>
      <td>325.355023</td>
    </tr>
    <tr>
      <th>6</th>
      <td>denver</td>
      <td>125.033447</td>
    </tr>
    <tr>
      <th>7</th>
      <td>indianapolis</td>
      <td>94.546119</td>
    </tr>
    <tr>
      <th>8</th>
      <td>los angeles</td>
      <td>582.602740</td>
    </tr>
    <tr>
      <th>9</th>
      <td>miami</td>
      <td>255.173630</td>
    </tr>
    <tr>
      <th>10</th>
      <td>rockville</td>
      <td>252.000571</td>
    </tr>
    <tr>
      <th>11</th>
      <td>nashville</td>
      <td>85.804452</td>
    </tr>
    <tr>
      <th>12</th>
      <td>newark</td>
      <td>1207.603425</td>
    </tr>
    <tr>
      <th>13</th>
      <td>new york</td>
      <td>1207.603425</td>
    </tr>
    <tr>
      <th>14</th>
      <td>alexandria</td>
      <td>551.476256</td>
    </tr>
    <tr>
      <th>15</th>
      <td>philadelphia</td>
      <td>252.000571</td>
    </tr>
    <tr>
      <th>16</th>
      <td>pittsburgh</td>
      <td>109.564612</td>
    </tr>
    <tr>
      <th>17</th>
      <td>raleigh</td>
      <td>55.625000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>washington</td>
      <td>551.476256</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Extracting the data we need from the commuter public tranport data
list_of_df = []
for index,row in cities_df.iterrows():
    ge = commuter_public_df[commuter_public_df["City"]==row["City"]]
    ga = ge[ge["State"].str.contains(row["State"])][["City","State","Mode","Average Speed (mi/hr)","Passengers per Hour","Average Passenger Trip Length (mi)","Directional Route Miles"]]
    list_of_df.append(ga)

```


```python
##Combining the data frame into one
merged_df = pd.concat(list_of_df)
merged_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>State</th>
      <th>Mode</th>
      <th>Average Speed (mi/hr)</th>
      <th>Passengers per Hour</th>
      <th>Average Passenger Trip Length (mi)</th>
      <th>Directional Route Miles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>110</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>MB</td>
      <td>12.33</td>
      <td>29.75</td>
      <td>4.25</td>
      <td>1672.2</td>
    </tr>
    <tr>
      <th>111</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>HR</td>
      <td>26.56</td>
      <td>85.81</td>
      <td>6.63</td>
      <td>96.1</td>
    </tr>
    <tr>
      <th>112</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>DR</td>
      <td>15.40</td>
      <td>1.58</td>
      <td>10.91</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>113</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>DR</td>
      <td>17.47</td>
      <td>1.71</td>
      <td>13.16</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>314</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>VP</td>
      <td>42.11</td>
      <td>6.87</td>
      <td>41.16</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Changing data types to floats so that calculations can be made
merged_df["Average Speed (mi/hr)"] = merged_df["Average Speed (mi/hr)"].astype(float)
```


```python
##This is a pain, but we need this to get rid of the rows that have a -  in the cell
##YOU NEED THIS PART
part1 = ~merged_df["Average Passenger Trip Length (mi)"].str.contains("-")

```


```python
##Concating the true false
list2 = [merged_df,part1]
merged_df1 = pd.concat(list2)
merged_df["This"] = part1
```


```python
merged_df2 = merged_df[merged_df["This"]==True]
```


```python
del merged_df2["This"]
```


```python
merged_df2.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>State</th>
      <th>Mode</th>
      <th>Average Speed (mi/hr)</th>
      <th>Passengers per Hour</th>
      <th>Average Passenger Trip Length (mi)</th>
      <th>Directional Route Miles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>110</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>MB</td>
      <td>12.33</td>
      <td>29.75</td>
      <td>4.25</td>
      <td>1672.2</td>
    </tr>
    <tr>
      <th>111</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>HR</td>
      <td>26.56</td>
      <td>85.81</td>
      <td>6.63</td>
      <td>96.1</td>
    </tr>
    <tr>
      <th>112</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>DR</td>
      <td>15.40</td>
      <td>1.58</td>
      <td>10.91</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>113</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>DR</td>
      <td>17.47</td>
      <td>1.71</td>
      <td>13.16</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>314</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>VP</td>
      <td>42.11</td>
      <td>6.87</td>
      <td>41.16</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Setting more data types for calcuations
merged_df2[["Average Passenger Trip Length (mi)","Average Speed (mi/hr)"]] = merged_df2[["Average Passenger Trip Length (mi)","Average Speed (mi/hr)"]].astype(float)
```


```python
##Calculating the average trip length

merged_df2['Average Trip Time'] = (merged_df2["Average Passenger Trip Length (mi)"]/merged_df2["Average Speed (mi/hr)"])*60
```


```python

##Mode Symbol:
##CB: Commuter Bus
##DR: Demand Response
##HR: Heavy Rail
##FB: Ferryboat
##MB: Bus
##CB: Commuter Bus
##DT: Demand Response Taxi (how NY has none of this..)
##CR: Commuter Rail
##VP: Vanpool
##LR: Light Rail
##SR: Streetcar Rail
##TB: Trolleybus
##RB: Bus Rapid Transit
##MG: Monorail/Automated Guideway
##YR: Hybrid Rail

##because new york sucks...
merged_df2 = merged_df2.set_value(1395, 'Passengers per Hour', 1165.53)
```


```python
##You need this too
##Setting mroe data types for calculations

merged_df2["Passengers per Hour"] = merged_df2["Passengers per Hour"].astype(float)

```


```python
# d1["Passengers per Hour"] = d1["Passengers per Hour"].astype(float)
# sum1 = d1["Passengers per Hour"].sum()

# d1["Percentage of Passengers utilized"] = d1["Passengers per Hour"]/sum1

# d1["weighted"] = d1["Percentage of Passengers utilized"]*d1["Average Trip Time"] 

# weighted_score = d1["weighted"].sum()

# d1

list_of_df1 = []

###Doing the commuter driving concatanation

for index,row in cities_df.iterrows():
    ge1 = commuter_df[commuter_df["City"].str.contains(row["City"])]
    ga1 = ge1[ge1["State"].str.contains(row["State"])][["City","State","Minutes"]]
    list_of_df1.append(ga1)

commmuting_merged_df = pd.concat(list_of_df1)

commmuting_merged_df
##Droppinng rows we do not need
commmuting_merged_df=commmuting_merged_df.drop(105)
commmuting_merged_df=commmuting_merged_df.drop(168)
commmuting_merged_df=commmuting_merged_df.drop(220)
commmuting_merged_df=commmuting_merged_df.drop(510)
commmuting_merged_df=commmuting_merged_df.drop(563)
commmuting_merged_df=commmuting_merged_df.drop(624)
commmuting_merged_df=commmuting_merged_df.drop(686)
# commmuting_merged_df=commmuting_merged_df.drop(921)


##Setting the cities equal to the correct string, so that they can be combined later

for index,row in cities_df.iterrows():
    for index3,row3 in commmuting_merged_df.iterrows():
        if commmuting_merged_df["City"].str.contains(row["City"])[index3]:
            commmuting_merged_df.set_value(index3,"City",row["City"])
```


```python
##These got mixed up for some reason...probably because they are in the same area and have the same value
commmuting_merged_df.set_value(921,"City","washington")
commmuting_merged_df.set_value(923,"State","VA")

#commmuting_merged_df["City"].str.contains(row["City"])[index3]
#cities_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>State</th>
      <th>Minutes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>46</th>
      <td>atlanta</td>
      <td>GA</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>55</th>
      <td>austin</td>
      <td>TX</td>
      <td>26.4</td>
    </tr>
    <tr>
      <th>106</th>
      <td>boston</td>
      <td>MA</td>
      <td>32.0</td>
    </tr>
    <tr>
      <th>167</th>
      <td>chicago</td>
      <td>IL</td>
      <td>31.3</td>
    </tr>
    <tr>
      <th>197</th>
      <td>columbus</td>
      <td>OH</td>
      <td>23.5</td>
    </tr>
    <tr>
      <th>219</th>
      <td>dallas</td>
      <td>TX</td>
      <td>27.8</td>
    </tr>
    <tr>
      <th>237</th>
      <td>denver</td>
      <td>CO</td>
      <td>27.3</td>
    </tr>
    <tr>
      <th>411</th>
      <td>indianapolis</td>
      <td>IN</td>
      <td>24.8</td>
    </tr>
    <tr>
      <th>512</th>
      <td>los angeles</td>
      <td>CA</td>
      <td>30.4</td>
    </tr>
    <tr>
      <th>565</th>
      <td>miami</td>
      <td>FL</td>
      <td>30.6</td>
    </tr>
    <tr>
      <th>922</th>
      <td>rockville</td>
      <td>MD</td>
      <td>34.6</td>
    </tr>
    <tr>
      <th>610</th>
      <td>nashville</td>
      <td>TN</td>
      <td>27.0</td>
    </tr>
    <tr>
      <th>627</th>
      <td>newark</td>
      <td>NJ</td>
      <td>32.7</td>
    </tr>
    <tr>
      <th>628</th>
      <td>new york</td>
      <td>NY</td>
      <td>37.1</td>
    </tr>
    <tr>
      <th>689</th>
      <td>philadelphia</td>
      <td>PA</td>
      <td>31.5</td>
    </tr>
    <tr>
      <th>697</th>
      <td>pittsburgh</td>
      <td>PA</td>
      <td>26.5</td>
    </tr>
    <tr>
      <th>727</th>
      <td>raleigh</td>
      <td>NC</td>
      <td>25.4</td>
    </tr>
    <tr>
      <th>921</th>
      <td>washington</td>
      <td>DC</td>
      <td>34.4</td>
    </tr>
    <tr>
      <th>923</th>
      <td>alexandria</td>
      <td>VA</td>
      <td>34.4</td>
    </tr>
  </tbody>
</table>
</div>




```python
##Doing a merge for average number of people driving per hour (sum_driving)
##And the main dataframe (merged_df2) ##The values will be repeated in merged_df7
merged_df7 = pd.merge(sum_driving_df,merged_df2,how="right")

##Do Another merge of that merge, and the average commuting time, again, values will be repeated
merged_df9 = pd.merge(merged_df7,commmuting_merged_df, on="City",how="right")

##This is testing for the For loop
# merged_df9[merged_df9["City"]=="atlanta"]["Minutes"][0]
# merged_df9[merged_df9["City"]=="atlanta"]["Average People per Hour"][0]

merged_df9.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Average People per Hour</th>
      <th>State_x</th>
      <th>Mode</th>
      <th>Average Speed (mi/hr)</th>
      <th>Passengers per Hour</th>
      <th>Average Passenger Trip Length (mi)</th>
      <th>Directional Route Miles</th>
      <th>Average Trip Time</th>
      <th>State_y</th>
      <th>Minutes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>atlanta</td>
      <td>244.677511</td>
      <td>GA</td>
      <td>MB</td>
      <td>12.33</td>
      <td>29.75</td>
      <td>4.25</td>
      <td>1672.2</td>
      <td>20.681265</td>
      <td>GA</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>atlanta</td>
      <td>244.677511</td>
      <td>GA</td>
      <td>HR</td>
      <td>26.56</td>
      <td>85.81</td>
      <td>6.63</td>
      <td>96.1</td>
      <td>14.977410</td>
      <td>GA</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>atlanta</td>
      <td>244.677511</td>
      <td>GA</td>
      <td>DR</td>
      <td>15.40</td>
      <td>1.58</td>
      <td>10.91</td>
      <td>0.0</td>
      <td>42.506494</td>
      <td>GA</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>atlanta</td>
      <td>244.677511</td>
      <td>GA</td>
      <td>DR</td>
      <td>17.47</td>
      <td>1.71</td>
      <td>13.16</td>
      <td>0.0</td>
      <td>45.197481</td>
      <td>GA</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>atlanta</td>
      <td>244.677511</td>
      <td>GA</td>
      <td>VP</td>
      <td>42.11</td>
      <td>6.87</td>
      <td>41.16</td>
      <td>0.0</td>
      <td>58.646402</td>
      <td>GA</td>
      <td>31.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
##We do not need this column
del merged_df9["State_y"]
```


```python
##This got a lot more complicated, this creates a two dictionaries of weights. The first one considers just public transportation. The second one incorporates commmuting driving as a factor
##The reason why it is separated is because it is apparant that most people drive to work. So the commuting driving heavily affects the overall score. 
weighted_scores_with_drive = {}
weighted_scores_without_drive_dict = {}
d1_list = []
d2_list = []
list_of_weighteddf = []
for index1,row1 in cities_df.iterrows():
    for index2,row2 in merged_df2.iterrows():
        d1 = []
        d2 = []
        d1 = merged_df2[merged_df2["City"]==row1["City"]]
        d2 = merged_df2[merged_df2["City"]==row1["City"]]
        sum2 = d2["Passengers per Hour"].sum()
        sum1 = d1["Passengers per Hour"].sum()+merged_df9[merged_df9["City"]==row1["City"]]["Average People per Hour"][merged_df9.index[merged_df9["City"]==row1["City"]][0]]
        d2["Percentage of Passengers utilized"] = d2["Passengers per Hour"]/sum2
        d1["Percentage of Passengers utilized"] = d1["Passengers per Hour"]/sum1
        d2["weighted"] = d2["Percentage of Passengers utilized"]*d2["Average Trip Time"]
        d1.set_value(index2+1,"Percentage of Passengers utilized",merged_df9[merged_df9["City"]==row1["City"]]["Average People per Hour"][merged_df9.index[merged_df9["City"]==row1["City"]][0]]/sum1)
        d1["weighted"] = d1["Percentage of Passengers utilized"]*d1["Average Trip Time"]
        d1.set_value(index2+1,"weighted", merged_df9[merged_df9["City"]==row1["City"]]["Average People per Hour"][merged_df9.index[merged_df9["City"]==row1["City"]][0]]/sum1*merged_df9[merged_df9["City"]==row1["City"]]["Minutes"][merged_df9.index[merged_df9["City"]==row1["City"]][0]])
        weighted_score_with_drive = d1["weighted"].sum()
        weighted_score_wo_drive = d2["weighted"].sum()
        weighted_scores_with_drive[row1["City"]] = weighted_score_with_drive
        weighted_scores_without_drive_dict[row1["City"]]= weighted_score_wo_drive
        d1_list.append(d1)
        d2_list.append(d2)
        list_of_weighteddf.append(d1)

```


```python
##More Testing for the for loop
#merged_df9.index[merged_df9["City"]=="chicago"][0]
```


```python
#merged_df7
#commmuting_merged_df
sum_driving_df
df_for_graph = merged_df2.groupby("City")

sums_people_transport = pd.DataFrame(df_for_graph["Passengers per Hour"].sum())
sums_people_transport.reset_index(level=0, inplace=True)
sums_people_transport = sums_people_transport.rename(columns = {"index":"City"})
sums_people_transport.head()

len(sums_people_transport)

sums_people_transport

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Passengers per Hour</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>alexandria</td>
      <td>83.72</td>
    </tr>
    <tr>
      <th>1</th>
      <td>atlanta</td>
      <td>197.84</td>
    </tr>
    <tr>
      <th>2</th>
      <td>austin</td>
      <td>111.77</td>
    </tr>
    <tr>
      <th>3</th>
      <td>boston</td>
      <td>531.14</td>
    </tr>
    <tr>
      <th>4</th>
      <td>chicago</td>
      <td>155.14</td>
    </tr>
    <tr>
      <th>5</th>
      <td>columbus</td>
      <td>25.22</td>
    </tr>
    <tr>
      <th>6</th>
      <td>dallas</td>
      <td>191.11</td>
    </tr>
    <tr>
      <th>7</th>
      <td>denver</td>
      <td>149.50</td>
    </tr>
    <tr>
      <th>8</th>
      <td>indianapolis</td>
      <td>19.43</td>
    </tr>
    <tr>
      <th>9</th>
      <td>los angeles</td>
      <td>484.22</td>
    </tr>
    <tr>
      <th>10</th>
      <td>miami</td>
      <td>189.05</td>
    </tr>
    <tr>
      <th>11</th>
      <td>nashville</td>
      <td>108.55</td>
    </tr>
    <tr>
      <th>12</th>
      <td>new york</td>
      <td>1875.43</td>
    </tr>
    <tr>
      <th>13</th>
      <td>newark</td>
      <td>406.25</td>
    </tr>
    <tr>
      <th>14</th>
      <td>philadelphia</td>
      <td>325.95</td>
    </tr>
    <tr>
      <th>15</th>
      <td>pittsburgh</td>
      <td>192.25</td>
    </tr>
    <tr>
      <th>16</th>
      <td>raleigh</td>
      <td>70.03</td>
    </tr>
    <tr>
      <th>17</th>
      <td>rockville</td>
      <td>24.12</td>
    </tr>
    <tr>
      <th>18</th>
      <td>washington</td>
      <td>175.67</td>
    </tr>
  </tbody>
</table>
</div>




```python

ind = np.arange(len(sum_driving_df))  # the x locations for the groups
width = 0.35       # the width of the bars

fig, ax = plt.subplots(figsize=(28, 10))
rects1 = ax.bar(ind, sums_people_transport["Passengers per Hour"], width, color='g')

rects2 = ax.bar(ind + width, sum_driving_df["Average People per Hour"], width, color='r')

# add some text for labels, title and axes ticks
ax.set_ylabel('Number of Passengers')
ax.set_title('Passengers by Commute Type')
ax.set_xticks(ind + width / 2)
ax.set_xticklabels(sums_people_transport["City"])

ax.legend((rects1[0], rects2[0]), ('Public Transport', 'Driving'))


def autolabel(rects):
    """
    Attach a text label above each bar displaying its height
    """
    for rect in rects:
        height = rect.get_height()
        ax.text(rect.get_x() + rect.get_width()/2., 1.05*height,
                '%d' % int(height),
                ha='center', va='bottom')

autolabel(rects1)
autolabel(rects2)
# # Sets the y limits of the current chart
plt.ylim(0,2100)
plt.xticks(rotation=90)
plt.savefig("Passengers by Commute Type")
plt.show()

# plt.bar(merged_df_rank_with_drive['City'], merged_df_rank_with_drive['Score with Driving'], color='b', alpha=0.5, align="center")



# # Give our chart some labels and a tile
# plt.title("City Trip Times")
# plt.xlabel("City")
# plt.ylabel("Average Trip Time (weighted)")

# # Print our chart to the screen
# plt.show()

#d1
#merged_df9[merged_df9["City"]==row1["City"]]["Average People per Hour"][merged_df9.index[merged_df9["City"]==row1["City"]][0]]
#index2.dtype
#merged_df9[merged_df9["City"]==row1["City"]]["Average People per Hour"][merged_df9.index[merged_df9["City"]==row1["City"]][0]]/sum1
#merged_df9[merged_df9["City"]==row1["City"]]["Average People per Hour"]
#merged_df9[merged_df9["City"]==row1["City"]]["Average People per Hour"][merged_df9.index[merged_df9["City"]==row1["City"]][0]]/sum1*merged_df9[merged_df9["City"]==row1["City"]]["Minutes"][merged_df9.index[merged_df9["City"]==row1["City"]][0]]
```


![png](output_42_0.png)



```python
weighted_scores_with_drive
```




    {'alexandria': 36.48303954956693,
     'atlanta': 26.689175339020807,
     'austin': 33.033972738811045,
     'boston': 24.47018951688783,
     'chicago': 30.058118822448613,
     'columbus': 24.55312216155005,
     'dallas': 28.254638253743792,
     'denver': 26.79359451892987,
     'indianapolis': 24.28920990648313,
     'los angeles': 26.692047319115282,
     'miami': 25.566191695281486,
     'nashville': 33.520134553976156,
     'new york': 30.64052599570794,
     'newark': 30.092175069311324,
     'philadelphia': 24.00981086106867,
     'pittsburgh': 17.166834197877012,
     'raleigh': 20.151047718865293,
     'rockville': 33.04361892625478,
     'washington': 29.697949825493055}




```python
weighted_scores_without_drive_dict
```




    {'alexandria': 50.20433495432511,
     'atlanta': 21.35778709500006,
     'austin': 38.28157924159262,
     'boston': 21.374400390113678,
     'chicago': 26.851125785357873,
     'columbus': 28.64023462741277,
     'dallas': 29.028636680967267,
     'denver': 26.370065267255523,
     'indianapolis': 21.803712180111916,
     'los angeles': 22.230721081748168,
     'miami': 18.77171860071003,
     'nashville': 38.67404125805223,
     'new york': 26.481222300518358,
     'newark': 22.340253057725,
     'philadelphia': 18.21894742117664,
     'pittsburgh': 11.847798106079916,
     'raleigh': 15.981792105012396,
     'rockville': 16.782884310618066,
     'washington': 14.936926176679231}




```python
merged_df_rank_with_drive = pd.DataFrame.from_dict(weighted_scores_with_drive, orient = "index")


merged_df_rank_with_drive = merged_df_rank_with_drive.rename(columns={0: 'Score with Driving'})
merged_df_rank_with_drive.reset_index(level=0, inplace=True)
merged_df_rank_with_drive = merged_df_rank_with_drive.rename(columns = {"index":"City"})

merged_df_rank_with_drive.sort_values("Score with Driving")
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Score with Driving</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>16</th>
      <td>pittsburgh</td>
      <td>17.166834</td>
    </tr>
    <tr>
      <th>17</th>
      <td>raleigh</td>
      <td>20.151048</td>
    </tr>
    <tr>
      <th>15</th>
      <td>philadelphia</td>
      <td>24.009811</td>
    </tr>
    <tr>
      <th>7</th>
      <td>indianapolis</td>
      <td>24.289210</td>
    </tr>
    <tr>
      <th>2</th>
      <td>boston</td>
      <td>24.470190</td>
    </tr>
    <tr>
      <th>4</th>
      <td>columbus</td>
      <td>24.553122</td>
    </tr>
    <tr>
      <th>9</th>
      <td>miami</td>
      <td>25.566192</td>
    </tr>
    <tr>
      <th>0</th>
      <td>atlanta</td>
      <td>26.689175</td>
    </tr>
    <tr>
      <th>8</th>
      <td>los angeles</td>
      <td>26.692047</td>
    </tr>
    <tr>
      <th>6</th>
      <td>denver</td>
      <td>26.793595</td>
    </tr>
    <tr>
      <th>5</th>
      <td>dallas</td>
      <td>28.254638</td>
    </tr>
    <tr>
      <th>18</th>
      <td>washington</td>
      <td>29.697950</td>
    </tr>
    <tr>
      <th>3</th>
      <td>chicago</td>
      <td>30.058119</td>
    </tr>
    <tr>
      <th>12</th>
      <td>newark</td>
      <td>30.092175</td>
    </tr>
    <tr>
      <th>13</th>
      <td>new york</td>
      <td>30.640526</td>
    </tr>
    <tr>
      <th>1</th>
      <td>austin</td>
      <td>33.033973</td>
    </tr>
    <tr>
      <th>10</th>
      <td>rockville</td>
      <td>33.043619</td>
    </tr>
    <tr>
      <th>11</th>
      <td>nashville</td>
      <td>33.520135</td>
    </tr>
    <tr>
      <th>14</th>
      <td>alexandria</td>
      <td>36.483040</td>
    </tr>
  </tbody>
</table>
</div>




```python
merged_df_rank_wo_drive = pd.DataFrame.from_dict(weighted_scores_without_drive_dict, orient = "index")

merged_df_rank_wo_drive = merged_df_rank_wo_drive.rename(columns={0: 'Score without Driving'})
merged_df_rank_wo_drive.reset_index(level=0, inplace=True)
merged_df_rank_wo_drive = merged_df_rank_wo_drive.rename(columns = {"index":"City"})

merged_df_rank_wo_drive.sort_values("Score without Driving")
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Score without Driving</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>16</th>
      <td>pittsburgh</td>
      <td>11.847798</td>
    </tr>
    <tr>
      <th>18</th>
      <td>washington</td>
      <td>14.936926</td>
    </tr>
    <tr>
      <th>17</th>
      <td>raleigh</td>
      <td>15.981792</td>
    </tr>
    <tr>
      <th>10</th>
      <td>rockville</td>
      <td>16.782884</td>
    </tr>
    <tr>
      <th>15</th>
      <td>philadelphia</td>
      <td>18.218947</td>
    </tr>
    <tr>
      <th>9</th>
      <td>miami</td>
      <td>18.771719</td>
    </tr>
    <tr>
      <th>0</th>
      <td>atlanta</td>
      <td>21.357787</td>
    </tr>
    <tr>
      <th>2</th>
      <td>boston</td>
      <td>21.374400</td>
    </tr>
    <tr>
      <th>7</th>
      <td>indianapolis</td>
      <td>21.803712</td>
    </tr>
    <tr>
      <th>8</th>
      <td>los angeles</td>
      <td>22.230721</td>
    </tr>
    <tr>
      <th>12</th>
      <td>newark</td>
      <td>22.340253</td>
    </tr>
    <tr>
      <th>6</th>
      <td>denver</td>
      <td>26.370065</td>
    </tr>
    <tr>
      <th>13</th>
      <td>new york</td>
      <td>26.481222</td>
    </tr>
    <tr>
      <th>3</th>
      <td>chicago</td>
      <td>26.851126</td>
    </tr>
    <tr>
      <th>4</th>
      <td>columbus</td>
      <td>28.640235</td>
    </tr>
    <tr>
      <th>5</th>
      <td>dallas</td>
      <td>29.028637</td>
    </tr>
    <tr>
      <th>1</th>
      <td>austin</td>
      <td>38.281579</td>
    </tr>
    <tr>
      <th>11</th>
      <td>nashville</td>
      <td>38.674041</td>
    </tr>
    <tr>
      <th>14</th>
      <td>alexandria</td>
      <td>50.204335</td>
    </tr>
  </tbody>
</table>
</div>




```python
merged_df_rank_with_drive_max = max(merged_df_rank_with_drive["Score with Driving"])
merged_df_rank_with_drive_min = min(merged_df_rank_with_drive["Score with Driving"])
merged_df_rank_with_drive_range = merged_df_rank_with_drive_max-merged_df_rank_with_drive_min
merged_df_rank_with_drive["Degree_score"]=1.0
```


```python
for index,row in merged_df_rank_with_drive.iterrows():
    score = abs((row["Score with Driving"]-merged_df_rank_with_drive_max)/merged_df_rank_with_drive_range*-10)
    score = round(score,1)
    merged_df_rank_with_drive.set_value(index,"Degree_score",score)
```


```python
for index,row in merged_df_rank_with_drive.iterrows():
    city = row["City"]
    state = cities_df[cities_df["City"]==city]["State"][cities_df.index[cities_df["City"]==city][0]]
    merged_df_rank_with_drive.set_value(index,"State",state)
```


```python
merged_df_rank_with_drive.sort_values("Degree_score",ascending=False)
```


```python
merged_df_rank_with_drive.to_csv("rankings_trasport_driving.csv")
```


```python

plt.bar(merged_df_rank_with_drive['City'], merged_df_rank_with_drive['Score with Driving'], color='b', alpha=0.5, align="center")

# Sets the y limits of the current chart
plt.ylim(0,50)

# Give our chart some labels and a tile
plt.title("City Trip Times")
plt.xlabel("City")
plt.ylabel("Average Trip Time (weighted)")
plt.xticks(rotation=90)
plt.savefig("City Trip Times")
# Print our chart to the screen
plt.show()
```


![png](output_52_0.png)



```python
plt.bar(merged_df_rank_with_drive['City'], merged_df_rank_wo_drive['Score without Driving'], color='g', alpha=0.5, align="center")

# Sets the y limits of the current chart
#plt.ylim(0,5)

# Give our chart some labels and a tile
plt.title("City Trip Times (Public Transportation Only)")
plt.xlabel("City")
plt.ylabel("Average Trip Time (weighted)")
plt.xticks(rotation=90)
plt.savefig("City Trip Times Public Transport")
# Print our chart to the screen
plt.show()
```


![png](output_53_0.png)

