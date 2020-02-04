How much pizza did the Dominos on State & Packard sell last year?
According to Dominos, the average franchise in the continental US made approximately $60000 in revenue per month. Dividing  60000by30givesusanaveragedailyincomeof$2000.Assumingthat80%ofDomino′srevenuecomesfrompizza,thatmeansthatonaverage,storesgenerateabout 1,600 of revenue from pizza. Since according to realmenuprices.com, the average pizza at Dominos costs $12.50, the average store sells 1,600 / 12.5 = 128 pizzas per day. Using this as our baseline, we can begin to make other assumptions:

ASSUMPTIONS:

General College Weight: 1.2, college stores see generally 20% more demand than average stores
College Holiday Sensitivity: 0.5, college students are 50% less likely to buy pizza on national holidays.
College Break Sensitivity: 0.4, college students are 60% less likely to buy pizza during school breaks.
College Gameday Sensitivity: 5, college students are five times more likely to buy pizza on gamedays
import pandas as pd
import numpy as np
from matplotlib import pyplot as plt
import datetime as dt
import math

import warnings
warnings.filterwarnings('ignore')
pd.set_option('display.max_rows', 50)
currentday = dt.date(2019, 1, 1)

days = []
for i in range(365):
    days.append(currentday) 
    currentday += dt.timedelta(days = 1)
days=np.array(days)

signal = np.array([1 for i in range(365)])

pizza = pd.DataFrame({"date":days, "signal":signal})
#applying assumptions

# 1. General College Weight
pizza["signal"] = pizza["signal"] * 1.2

# 2. Holiday Sensitivity
import holidays
us_hols = holidays.US()
pizza["public holidays"] = pizza["date"].apply(lambda date: date in us_hols)

# 3. College Breaks
summer = [dt.date(2019, 4, 30) + dt.timedelta(days = i) for i in range((dt.date(2019, 9, 2)-dt.date(2019, 4, 30)).days)]
winter1 = [dt.date(2019, 12, 15) + dt.timedelta(days = i) for i in range((dt.date(2019, 12, 31)-dt.date(2019, 12, 15)).days)]
winter2 = [dt.date(2019, 1, 1) + dt.timedelta(days = i) for i in range(8)]
breaks = np.array(summer + winter1 + winter2)
pizza["breaks"] = pizza["date"].apply(lambda date: date in breaks)

# 4. Gamedays
# assume occur on days where school is in session and is on saturday
saturdays = [dt.date(2019, 1, 5) + dt.timedelta(days = i*7) for i in range(52)]

def gameday(date):
    if date not in breaks and date in saturdays:
        return True
    else:
        return False

pizza["gamedays"] = pizza["date"].apply(gameday)


# changing signal:
index = 0
for i in pizza["public holidays"]:
    if i==True:
        pizza["signal"][index] *= 0.5 
    index += 1
    
index = 0
for i in pizza["breaks"]:
    if i==True:
        pizza["signal"][index] *= 0.4 
    index += 1
    
index = 0
for i in pizza["gamedays"]:
    if i==True:
        pizza["signal"][index] *= 5 
    index += 1

pizza
date	signal	public holidays	breaks	gamedays
0	2019-01-01	0.24	True	True	False
1	2019-01-02	0.48	False	True	False
2	2019-01-03	0.48	False	True	False
3	2019-01-04	0.48	False	True	False
4	2019-01-05	0.48	False	True	False
...	...	...	...	...	...
360	2019-12-27	0.48	False	True	False
361	2019-12-28	0.48	False	True	False
362	2019-12-29	0.48	False	True	False
363	2019-12-30	0.48	False	True	False
364	2019-12-31	1.20	False	False	False
365 rows × 5 columns

total_pizzas = pizza["signal"].sum() * 128
print("Estimated Pizzas Sold in 2019: ",math.ceil(total_pizzas))
Estimated Pizzas Sold in 2019:  60795
Estimated Total Pizzas Sold In 2019 by the Domino's on State & Packard:
60795
