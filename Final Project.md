```python
# Making a function that switches independent variables for plots on Juptyer notebooks
```


```python
# Example with simple data

import matplotlib.pyplot as plt
import numpy as np
from ipywidgets import interact, fixed

my_x1 = np.array([1, 2, 3, 4])
my_y1 = np.array([9, 10, 11, 12])
my_y2 = np.array([8, 9, 10, 11])

y_lib = [my_y1, my_y2]

def make_plot(my_x, y_lib, index):
    
    fig,ax = plt.subplots(1,1)
    ax.set_xlabel('my_x')
    ax.set_ylabel('my_y')
    ax.plot(my_x, y_lib[index])
    
def interactive_plot():
    interact(make_plot, my_x=fixed(my_x1), y_lib=fixed(y_lib), index=(0,1))
    
```


```python
interactive_plot()
```


    interactive(children=(IntSlider(value=0, description='index', max=1), Output()), _dom_classes=('widget-interac…



```python
# Example with astronomical data below
```


```python
import pandas as pd
import sqlite3
```


```python
import os
try:
    os.remove("msdss.db")
except OSError:
    pass
```


```python
con = sqlite3.connect("msdss.db")
```


```python
con.execute("""
CREATE TABLE `sources` (
    `run`      INTEGER,
    `rerun`    INTEGER,
    `camcol`   INTEGER,
    `field`    INTEGER,
    `obj`      INTEGER,
    `type`     INTEGER,

    `ra`          REAL,
    `dec`         REAL,
    `psfMag_r`    REAL,
    `psfMag_g`    REAL,
    `psfMagErr_r` REAL,
    `psfMagErr_g` REAL
);
""")
```




    <sqlite3.Cursor at 0x7f5bfcb78a40>




```python
con.execute("""
CREATE TABLE `runs` (
    `run`         INTEGER,
    `ra`          REAL,
    `dec`         REAL,
    `mjdstart`    REAL,
    `mjdend`      REAL,
    `node`        REAL,
    `inclination` REAL,
    `mu0`         REAL,
    `nu0`         REAL
);
""")
```




    <sqlite3.Cursor at 0x7f5bfbed3c00>




```python
runs = pd.read_csv('runs.txt', 
            sep=" ", header=None, skiprows=1, 
            names=['run', 'ra', 'dec', 'mjdstart', 'mjdend', 'node', 'inclination', 'mu0', 'nu0'],
            index_col = 'run')
```


```python
sources = pd.read_csv('sample.csv',
                        dtype={
                                'run': np.int16,
                                'rerun': np.int16,
                                'camcol': np.int8,
                                'field': np.int16,
                                'obj':  np.int32,
                                'type': np.int16,
                                'psfMag_r': np.float32,
                                'psfMag_g': np.float32,
                                'psfMagErr_r': np.float32,
                                'psfMagErr_g': np.float32,
                              },
                       index_col=['run', 'rerun', 'camcol', 'field', 'obj'],
                       na_values={
                            'psfMagErr_g': ["-9999"],
                            'psfMagErr_r': ["-9999"],
                            'psfMag_g': ["-9999"],
                            'psfMag_r': ["-9999"],
                       },
                       verbose=True
                     )
```

    Tokenization took: 31.23 ms
    Type conversion took: 27.32 ms
    Parser memory cleanup took: 0.01 ms



```python
runs.to_sql('runs', con, if_exists='replace')
```


```python
sources.to_sql('sources', con, if_exists='replace')
```


```python
result = pd.read_sql("""
    SELECT
        sources.ra, sources.dec, sources.run, mjdstart, psfMag_r, psfMag_g
    FROM
        sources JOIN runs ON sources.run = runs.run
""", con)
```


```python
result[:5]
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
      <th>ra</th>
      <th>dec</th>
      <th>run</th>
      <th>mjdstart</th>
      <th>psfMag_r</th>
      <th>psfMag_g</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8.129444</td>
      <td>26.626617</td>
      <td>7757</td>
      <td>54764.323971</td>
      <td>17.048889</td>
      <td>18.165350</td>
    </tr>
    <tr>
      <th>1</th>
      <td>8.127839</td>
      <td>26.627246</td>
      <td>7757</td>
      <td>54764.323971</td>
      <td>17.374020</td>
      <td>17.928749</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8.127323</td>
      <td>26.625120</td>
      <td>7757</td>
      <td>54764.323971</td>
      <td>20.146601</td>
      <td>21.352970</td>
    </tr>
    <tr>
      <th>3</th>
      <td>24.516117</td>
      <td>-1.165794</td>
      <td>4288</td>
      <td>52971.187293</td>
      <td>22.970320</td>
      <td>24.325899</td>
    </tr>
    <tr>
      <th>4</th>
      <td>24.517941</td>
      <td>-1.179207</td>
      <td>4288</td>
      <td>52971.187293</td>
      <td>22.620520</td>
      <td>25.091089</td>
    </tr>
  </tbody>
</table>
</div>




```python

# Picking out Andromeda's coordinates of approximately 10.6 ra and 41 dec

result_of_interest = result.query('ra < 11').query('ra > 10.5').query('dec > 40').query('dec < 42')

my_x1 = result_of_interest['ra']
my_y1 = result_of_interest['psfMag_r']
my_y2 = result_of_interest['psfMag_g']

y_lib = [my_y1, my_y2]

def make_plot(my_x, y_lib, index):
    
    fig,ax = plt.subplots(1,1)
    ax.set_xlabel('my_x')
    ax.set_ylabel('my_y')
    ax.plot(my_x, y_lib[index])
    
def interactive_plot():
    interact(make_plot, my_x=fixed(my_x1), y_lib=fixed(y_lib), index=(0,1))
```


```python
# This plots the r and g bands of Andromeda

interactive_plot()
```


    interactive(children=(IntSlider(value=0, description='index', max=1), Output()), _dom_classes=('widget-interac…



```python

```
