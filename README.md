
# coding: utf-8

# # WeatherPy

# ### Analysis

# -High humidity appears to occur in cities located between 40-60 latitude

# -There is an exponential decrease in temperature from latitudes 20 to 60.  For every 20 degrees in latitude, temperature decreases by 20 degrees farenheit.

# -Cities in Latitudes 40-60, appear to have the highest wind speed.

# In[1]:


#import dependencies
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from citipy import citipy
import requests as req
import json
import time

#import API keys
from localenv import wkey


# In[2]:


#Build URL
url = "http://api.openweathermap.org/data/2.5/weather?"
units = "imperial" 
appid = wkey
settings = {"units": "imperial", "appid": wkey}
query_url = f"{url}appid={wkey}&units={units}&q="


# In[3]:


query_url


# In[4]:


#create empty lists
citylist = []
count = 0
dup = 'no'

for x in range(-90,90,1):
    for y in range(-180,180,1):
        city = citipy.nearest_city(x, y)
        citdict = {}
        citdict['city'] = city.city_name
        citdict['country'] = city.country_code
        citdict['lat'] = x
        citdict['long'] = y
        if len(citylist) == 0:
            citylist.append(citdict)
            count+=1
            continue
        else:
            #Get rid of duplicates
            for city in citylist:
                if city['city'] == citdict['city']:
                    dup = 'yes'
        if dup == 'no':
            citylist.append(citdict)
            count+=1
        else:
            dup = 'no'

print(len(citylist))


# In[5]:


print(citylist[0])


# In[6]:


#Create dataframe. Grab 500 random cities
citypd = pd.DataFrame({
    'city': [x['city'] for x in citylist],
    'country': [x['country'] for x in citylist],
})

citypd.head()

samplecity = citypd.sample(550)


# In[7]:


count = 0
weather_json = []

for index,row in samplecity.iterrows():
    count+= 1
    query_url = url + "appid=" + wkey + "&units=" + units + "&q=" + row['city']
    try:
        weather_response = req.get(query_url)
        cityweather = weather_response.json()
        weather_json.append(cityweather)
        city1 = data.get("name")
        city.append(city1)
        country1 = data.get("sys").get("country")
        country.append(country1)
    except:
        #print(f"No data for this city: {row['city']}")
        print(f"Processing Record {count} | {row['city']}")
        print(query_url)

print("-"*100)
print("                Data Retrieval Complete")
print("-"*100)


# In[8]:


latitude = []
longtitude = []
temperature = []
humidity = []
cloudiness = []
wind_speed = []
city = []
country = []
max_temp = []


for data in weather_json:
    try:
        weather_response = req.get(query_url)
        cityweather = weather_response.json()
        lat1 = data.get("coord").get("lat")
        latitude.append(lat1)
        temp1 = data.get("main").get("temp")
        temperature.append(temp1)
        city1 = data.get("name")
        city.append(city1)
        country1 = data.get("sys").get("country")
        country.append(country1)
        humi1 = data.get("main").get("humidity")
        humidity.append(humi1)
        cld1 = data.get("clouds").get("all")
        cloudiness.append(cld1)
        wind1 = data.get("wind").get("speed")
        wind_speed.append(wind1)
        long1 = data.get("coord").get("lon")
        longtitude.append(long1)
        temp1 = data.get("main").get("temp_max")
        max_temp.append(temp1)
        
    except:
        pass

    continue

weather_dict = {"city":city, "Temperature (F)": temperature, "Latitude": latitude, "Longitude":longtitude, "Country":country,
                "Humidity": humidity, "Cloudiness":cloudiness,"Wind Speed":wind_speed, "Max_Temp":max_temp}
weather_df = pd.DataFrame(weather_dict)
weather_df.head()


# In[9]:


weather_df.count()


# In[10]:


weather_df.to_csv("WeatherPy.csv",encoding="utf-8",index=False)


# ### Latitude vs Temperature (F)

# In[11]:


date = time.strftime("%m/%d/%Y")
plt.scatter(weather_df["Latitude"], 
            weather_df["Max_Temp"])
plt.title(f" Latitude vs. Max Temperature (F) {date}")
plt.ylabel("Max Temperature (F)")
plt.xlabel("Latitude")
plt.xlim((-90,90))
plt.grid(True)
sns.set()
plt.savefig("Latitude vs Temperature (F).png")
plt.show()


# ### Latitude vs Humidity (%)

# In[12]:


plt.scatter(weather_df["Latitude"], 
            weather_df["Humidity"])
plt.title(f"Latitude vs. Humidity (%) {date}")
plt.ylabel("Humidity (%)")
plt.xlabel("Latitude")
plt.xlim((-90,90))
plt.grid(True)
sns.set()
plt.savefig("Latitude vs Humidity (%).png")
plt.show()


# ### Latitude vs. Cloudiness (%)

# In[13]:


plt.scatter(weather_df["Latitude"], 
            weather_df["Cloudiness"])
plt.title(f"Latitude vs. Cloudiness {date}")
plt.ylabel("Cloudiness (%)")
plt.xlabel("Latitude")
plt.xlim((-90,90))
plt.grid(True)
sns.set()
plt.savefig("Latitude vs Cloudiness (%).png")
plt.show()


# ### Latitude vs Wind Speed (mph)

# In[14]:


plt.scatter(weather_df["Latitude"], 
            weather_df["Wind Speed"])
plt.title(f"Latitude vs. Wind Speed {date}")
plt.ylabel("Wind Speed (MPH)")
plt.xlabel("Latitude")
plt.xlim((-90,90))
plt.grid(True)
sns.set()
plt.savefig("Latitude vs Wind Speed (MPH).png")
plt.show()

