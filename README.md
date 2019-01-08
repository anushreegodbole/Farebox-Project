# Farebox-Project
```
import pandas as pd
import numpy as np
from pandas.io.html import read_html
import pickle
import sqlite3
```

```
refresh= True 

if refresh:
    page = 'https://en.wikipedia.org/wiki/Farebox_recovery_ratio'
    wikitables = read_html(page)
    table= wikitables[1]
    pickle.dump(table,open("wiki_table.pkl","wb"))

else:
    table = pickle.load(open("wiki_table.pkl","r"))


    
cont = table[0][1:] #pandas allows for slicing
country = table[1][1:]
system = table[2][1:]
ratios = table[3][1:]
faresystem = table[4][1:]
#fare rate = table[5]
year = table[6][1:]


#Converting ratios
def CleanRatio(raw_ratio):
    s=raw_ratio.split("%")
    s[0] = float(s[0])
    return float(s[0]/100.00)
    
clean_ratios = []
for ratio in ratios:
    clean_ratios.append(CleanRatio(ratio))

#Writing to SQLite3
db_file = "rates.db"
conn = sqlite3.connect(db_file)


create_table_sql = """CREATE TABLE IF NOT EXISTS systems(
                                       continent REAL,
                                       country REAL,
                                       system REAL,
                                       ratio REAL,
                                       faresystem REAL,
                                       USDrates REAL,
                                       year REAL
                                        ); """

cur = conn.cursor()
cur.execute(create_table_sql)

#make string values for these columns lowercase
def Lower(strings):
    strings = str(strings)
    v = strings.lower()
    return v
    
clean_continent = []
for c in cont:
    clean_continent.append(Lower(c))
    
clean_countries = []
for c in country:
    clean_countries.append(Lower(c))
    
clean_system = []
for c in system:
    clean_system.append(Lower(c))

clean_faresystem = []
for c in faresystem:
    clean_faresystem.append(Lower(c))    

clean_year = []
for c in year:
    clean_year.append(c)
    
#attempt for clean_UDS_rates (unfinished)
def Clean_Fare_Systems(raw_fare_systems):

def remove_non_ascii_1(text):
    return ''.join(i for i in str(text) if ord(i)<128)
    
def CleanUSD_Rate(raw_rate):
    import re
    a= remove_non_ascii_1(raw_rate)
    numeric_const_pattern = '[-+]? (?: (?: \d* \. \d+ ) | (?: \d+ \.? ) )(?: [Ee] [+-]? \d+ ) ?'
    rx = re.compile(numeric_const_pattern, re.VERBOSE)
    return (rx.findall(a))
    
allrates=[]
for rate in fare_rate:
    allrates.append(str(CleanUSD_Rate(rate))
    
for i in range(len(allrates)):
    if len(allrates[i])>1:
        del allrates[i][1]
        
for i in range(len(allrates)):
    del allrates[i][1:]
```


#were actually converted on excel:
clean_USD_rates=[0.455,1.32,1.76,0,1.408,0,0.144,0.64,0.64,0.803,0.42,0,0,2.938,0,0,0,0,0,0,0,1.056,2.034,4.84,0,2.26,3.164,4.3,0,0,2.5,1.25,2.4375,2.65,2.25,2.5,4,2.5,2.5,1.5,2.25,1.75,5,5,1.75,1.6,2.25,2,2.4375,2.4375,2.75,2.75,2.25,2.75,2.25,2,2.7375,2.5,2,1.4,2.5,2.5,2.75,2.25,1.2,2.5,2,6,3.75,2.25,1.25,2.5,2.75,2.25,3.975,2.2125,2,1.875,0,3.024,0.108,2.7072,0,0,0,0]

    
#can zip lists and join them together using for t in zip()
for r in zip(clean_continent, clean_countries, clean_system, clean_ratios, clean_faresystem, clean_USD_rates, clean_year):
    sql = """Insert INTO systems VALUES ("%s", "%s", "%s", "%s", "%s", "%s","%s")""" % r
    cur.execute(sql)
    conn.commit()

'''

    

