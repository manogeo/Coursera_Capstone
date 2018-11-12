

```python
import numpy as np # library to handle data in a vectorized manner

import pandas as pd # library for data analsysis
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

import json # library to handle JSON files

!conda install -c conda-forge geopy --yes # uncomment this line if you haven't completed the Foursquare API lab
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# import k-means from clustering stage
from sklearn.cluster import KMeans

!conda install -c conda-forge folium=0.5.0 --yes # uncomment this line if you haven't completed the Foursquare API lab
import folium # map rendering library

from bs4 import BeautifulSoup
import requests

print('Libraries imported.')
```

    Solving environment: done
    
    # All requested packages already installed.
    
    Solving environment: done
    
    # All requested packages already installed.
    
    Libraries imported.



```python
# getting data from internet
wikipedia_link='https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M'
raw_wikipedia_page= requests.get(wikipedia_link).text

# using beautiful soup to parse the HTML/XML codes.
soup = BeautifulSoup(raw_wikipedia_page,'xml')
#print(soup.prettify())


```


```python
# extracting the raw table inside that webpage
table = soup.find('table')

Postcode      = []
Borough       = []
Neighbourhood = []

# print(table)

# extracting a clean form of the table
for tr_cell in table.find_all('tr'):
    
    counter = 1
    Postcode_var      = -1
    Borough_var       = -1
    Neighbourhood_var = -1
    
    for td_cell in tr_cell.find_all('td'):
        if counter == 1: 
            Postcode_var = td_cell.text
        if counter == 2: 
            Borough_var = td_cell.text
            tag_a_Borough = td_cell.find('a')
            
        if counter == 3: 
            Neighbourhood_var = str(td_cell.text).strip()
            tag_a_Neighbourhood = td_cell.find('a')
            
        counter +=1
        
    if (Postcode_var == 'Not assigned' or Borough_var == 'Not assigned' or Neighbourhood_var == 'Not assigned'): 
        continue
    try:
        if ((tag_a_Borough is None) or (tag_a_Neighbourhood is None)):
            continue
    except:
        pass
    if(Postcode_var == -1 or Borough_var == -1 or Neighbourhood_var == -1):
        continue
        
    Postcode.append(Postcode_var)
    Borough.append(Borough_var)
    Neighbourhood.append(Neighbourhood_var)
```


```python
unique_p = set(Postcode)
print('num of unique Postal codes:', len(unique_p))
Postcode_u      = []
Borough_u       = []
Neighbourhood_u = []


for postcode_unique_element in unique_p:
    p_var = ''; b_var = ''; n_var = ''; 
    for postcode_idx, postcode_element in enumerate(Postcode):
        if postcode_unique_element == postcode_element:
            p_var = postcode_element;
            b_var = Borough[postcode_idx]
            if n_var == '': 
                n_var = Neighbourhood[postcode_idx]
            else:
                n_var = n_var + ', ' + Neighbourhood[postcode_idx]
    Postcode_u.append(p_var)
    Borough_u.append(b_var)
    Neighbourhood_u.append(n_var)
```

    num of unique Postal codes: 84



```python
toronto_dict = {'Postcode':Postcode_u, 'Borough':Borough_u, 'Neighbourhood':Neighbourhood_u}
df_toronto = pd.DataFrame.from_dict(toronto_dict)
df_toronto.to_csv('toronto_part1.csv')
df_toronto.head(14)
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
      <th>Postcode</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M9A</td>
      <td>Etobicoke</td>
      <td>Islington Avenue</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M4H</td>
      <td>East York</td>
      <td>Thorncliffe Park</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M1B</td>
      <td>Scarborough</td>
      <td>Rouge, Malvern</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M9W</td>
      <td>Etobicoke</td>
      <td>Northwest</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M9L</td>
      <td>North York</td>
      <td>Humber Summit</td>
    </tr>
    <tr>
      <th>5</th>
      <td>M4Y</td>
      <td>Downtown Toronto</td>
      <td>Church and Wellesley</td>
    </tr>
    <tr>
      <th>6</th>
      <td>M9N</td>
      <td>York</td>
      <td>Weston</td>
    </tr>
    <tr>
      <th>7</th>
      <td>M3J</td>
      <td>North York</td>
      <td>Northwood Park, York University</td>
    </tr>
    <tr>
      <th>8</th>
      <td>M2H</td>
      <td>North York</td>
      <td>Hillcrest Village</td>
    </tr>
    <tr>
      <th>9</th>
      <td>M2J</td>
      <td>North York</td>
      <td>Henry Farm</td>
    </tr>
    <tr>
      <th>10</th>
      <td>M5S</td>
      <td>Downtown Toronto</td>
      <td>University of Toronto</td>
    </tr>
    <tr>
      <th>11</th>
      <td>M1T</td>
      <td>Scarborough</td>
      <td>Tam O'Shanter</td>
    </tr>
    <tr>
      <th>12</th>
      <td>M6L</td>
      <td>North York</td>
      <td>Maple Leaf Park</td>
    </tr>
    <tr>
      <th>13</th>
      <td>M1W</td>
      <td>Scarborough</td>
      <td>Steeles West</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_toronto.shape
```




    (84, 3)


