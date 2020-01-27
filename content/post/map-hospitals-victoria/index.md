---
title: "Interactive Map of Hospitals in Victoria using Python"
date: 2020-01-27T23:43:34+11:00
draft: false
authors:
- admin
categories : ["Data Science"]
tags : ["visualization","folium","python","Data Science","maps","melbourne"]
image:
  placement: 3
  caption: ""
  focal_point: ""
featured: true

type: post 
comments: true 
share: true
summary: "There are 325 hospitals in victoria. We will visualize them in the map such that some insights can be extracted."

---

There are 325 hospitals in victoria. We will visualize them in the map such that some insights can be extracted.

## Purpose of this post:
- Mark all the hospitals in victoria using folium marker
- Which localities have the highest number of hospitals? - barplot 
- Visualizing localities with the highest number of hospitals in a map using folium Circle.
- Creating heatmap for density of hospitals in the region.

## What is used?
- Pandas - to read, filter and merge the data.
- seaborn to plot Barplot.
- Python's folium- Marker, Circle and Heatmap. Folium is specifically used to visualize maps.

## Data Reference:
1. Hospital locations by vic.gov.in - [here](https://discover.data.vic.gov.au/dataset/hospital-locations-spatial) .
2. Postcode details provided by  Matthew Procter - [here](https://www.matthewproctor.com/australian_postcodes) .

## Links :
- Check out  my [Github for all the code in this post](https://github.com/nagarajbhat/map-hospitals-victoria). 
- Check out my [Youtube video on Interactive map for hospitals in Victoria](https://youtu.be/uZu8EZr4uYU). 
- Check out my [Interactive Map application here](https://nagarajbhat.com/app/map-hospitals-victoria.html).

## 1. Import dependencies

```python
import pandas as pd
import folium
import matplotlib.pyplot as plt
import seaborn as sns
```

## 2. Load the data
Hospital location data is provided by vic.gov.au [here](https://discover.data.vic.gov.au/dataset/hospital-locations-spatial)

```python
data = pd.read_csv('./data/hospital_locations.csv')
data.head()
```
![image](/img/map-hospitals-victoria/step2.png)
*Fig.1: Data of hospitals in victoria*


## 3. Exploring the data

- What are the number of public and private hospitals in Victoria?

```python
print("Number of public hospitals in victoria :" ,len(data[data["Type"] == "PUBLIC"]))
print("Number of private hospitals in victoria :" ,len(data[data["Type"] == "PRIVATE"]))
```
![image](/img/map-hospitals-victoria/step3_number.PNG)
*Fig.2: Number of public and private hospitals in victoria*


- Fields in the data:

```python
data.columns
```
![image](/img/map-hospitals-victoria/step3_columns.PNG)
*Fig.3: Fields in the data*

- We do not require all the fields in the data, so let us **filter** it

```python
df = data.filter(["Y","X","LabelName","Type","StreetNum","RoadName","RoadType","Postcode"])
df.head()
```


![image](/img/map-hospitals-victoria/step3_filter.PNG)
*Fig.4: Filtered data*

## 4. Mark all the hospitals in Victoria in a map using folium Marker

We will use folium marker to mark all (325) the hospitals in Victoria.

```python
#Create a map
m = folium.Map(location=[-37.0,144], control_scale=True, zoom_start=7,attr = "text some")
df_copy = df.copy()

# loop through data to create Marker for each hospital
for i in range(0,len(df_copy)):
    
    # html to be displayed in the popup 
    html="""
    <h4> Hospital name: </h4>""" + str(df_copy.iloc[i]['LabelName'])+ \
    """<h4>Type:</h4>""" + str(df_copy.iloc[i]['Type']) +" Hospital" +\
    """<h4>Address:</h4>""" + str(df_copy.iloc[i]['StreetNum'])+" "+ str(df_copy.iloc[i]['RoadName'])+" "+ str(df_copy.iloc[i]['RoadType'])
    
    #IFrame 
    iframe = folium.IFrame(html=html, width=200, height=300)
    popup = folium.Popup(iframe, max_width=2650)
    
    #If you do not want html in popup this would be the alternative code ->
    #popup = html("<b>HOSPITAL NAME:</b> "+str(df_copy.iloc[i]['LabelName'])+"<br/><br/><b>TYPE:</b> "+str(df_copy.iloc[i]['Type']) +" hospital"\
   # +"<br/><br/> <b>ADDRESS:</B> " + str(df_copy.iloc[i]['StreetNum'])+" "+ str(df_copy.iloc[i]['RoadName'])+" "+ str(df_copy.iloc[i]['RoadType'])),
    
    folium.Marker(
    location=[df_copy.iloc[i]['Y'], df_copy.iloc[i]['X']],
    popup=popup,
    tooltip=str(df_copy.iloc[i]['LabelName']),
    icon=folium.Icon(color='green',icon='medkit',prefix="fa"),
    ).add_to(m)
        
m.save("./files/hospitals.html")
m
```

[![image](/img/map-hospitals-victoria/hospitals.PNG)](/files/map-hospitals-victoria/hospitals.html)
*Fig.5: Hospitals in Victoria (click on the image for interactive map)*

- Custom icons can also be created using folium.features.CustomIcon
- In the dataframe, Y is latitude and X is longitude
- IFrame is a way you can use HTML in a popup since popup doesn't have HTML facility by default

folium.Marker arguments - 
- tooltip - adds hover functionality
- popup - string to be displayed when clicked upon the marker. We display more details here such as hospital type and address.
- icon - icon can be selected from font awesome, just mention the prefix 'fa'.


## 5. Postcodes with highest number of hospitals

```python
df_postcode = data.groupby(['Postcode'],as_index=False)
postcode_count = df_postcode.count().filter(["Postcode","FID"])
postcode_count = postcode_count.sort_values(by="FID",ascending=False)
postcode_count
```
![image](/img/map-hospitals-victoria/step5.PNG)
*Fig.6: postcodes with highest number of hospitals*

- Note that FID field here just indicates the count of hospitals for a given postcode


## 6. Merge this with postcode details
- The original dataframe does not contain postcode details such as locality name, latitude, and longitude based on a given postcode.
- Postcode details are obtained from [here](https://www.matthewproctor.com/australian_postcodes) by Matthew Procter.
- both the dataframes are merged based on the common key - postcode.

```python
australian_postcodes = pd.read_csv('./data/australian_postcodes.csv')
vic_postcodes = australian_postcodes[australian_postcodes["state"]=='VIC']
vic_postcodes = vic_postcodes.filter(["postcode","locality","lat",'long'])
vic_postcodes.head(5)
```
![image](/img/map-hospitals-victoria/step6_postcode.PNG)
*Fig.7: postcode details data*
	
- Merge - 

```python
df_merged  = postcode_count.merge(vic_postcodes,left_on='Postcode',right_on='postcode',how="inner")
df_merged = df_merged.dropna()
top_localities = df_merged.sort_values(by="FID",ascending=False)
top_localities
```
![image](/img/map-hospitals-victoria/step6_merge.PNG)
*Fig.8: Merged data*

## 7. Localities with most hospitals

- Note: There can be more than one locality under a single postcode.
- We will find out which localities have the highest number of hospitals

```python
top_localities_15 = top_localities.head(15)
ax = sns.barplot(top_localities_15['FID'],top_localities_15['locality'],hue=top_localities_15['postcode'])
ax.set(xlabel="count - number of hospitals",ylabel="locality",title="Localities with highest number of hospitals in Victoria")
ax
```

![image](/img/map-hospitals-victoria/step7_barplot.PNG)
*Fig.9: Localities with the most number of hospitals*

As we can see from the barplot above, The top 5 localities with most number of hospitals are  East melbourne,Seddon and SeddonWest,Footscray and Frankston.

## 8. Map of the number of hospitals in a locality 
- We will use the folium circle to create this map.
- the size of the circle represents the number of hospitals in the locality.

```python
from folium.plugins import HeatMap
from folium.features import CustomIcon
m = folium.Map(location=[-38.0,145], control_scale=True, zoom_start=10,attr = "text some",tiles='cartodbdark_matter')
df_copy = df_merged.copy()
df_copy['count'] = 1

# create Folium circle
for i in range(0,len(df_copy)):
     # html to be displayed in the popup 
    html="""
    <h4> Locality Name: </h4>""" + str(df_copy.iloc[i]['locality'])+\
    """<h4>Postcode:</h4>""" + str(df_copy.iloc[i]['postcode'])+\
    """<h4>Number of hospitals:</h4>""" + str(df_copy.iloc[i]['FID'])
    
    #IFrame 
    iframe = folium.IFrame(html=html, width=200, height=300)
    popup = folium.Popup(iframe, max_width=2650)
    
    folium.Circle(
    location=[df_copy.iloc[i]['lat'], df_copy.iloc[i]['long']],
    radius=float(df_copy.iloc[i]['FID']*200),
    popup=popup,
    tooltip=str(df_copy.iloc[i]['locality']),
    color='crimson',
     fill=True,
     fill_color='green'
    ).add_to(m)
        
m.save("./files/locality.html")
m
```

[![image](/img/map-hospitals-victoria/locality.png)](/files/map-hospitals-victoria/locality.html)
*Fig.10: Locality view - Indicating the number of Hospitals in each locality in Victoria (click on the image for interactive map)*

Disclaimer :
- Also note that the circle is a just a representation for the locality. For accurate hospital locations prefer hospitals view.

## 9. Heatmap of the hospitals

```python
m = folium.Map(location=[-38.0,145], control_scale=True, zoom_start=10,attr = "text some",tiles='cartodbdark_matter')
df_copy = df.copy()
df_copy['count']=1
HeatMap(data=df_copy[['Y', 'X', 'count']].groupby(['Y', 'X']).sum().reset_index().values.tolist(),gradient={0.2: 'blue', 0.4: 'lime', 0.6: 'orange', 1: 'red'},radius=15, max_zoom=13).add_to(m)
m.save("./files/heatmap.html")
m

```
[![image](/img/map-hospitals-victoria/heatmap.PNG)](/files/map-hospitals-victoria/heatmap.html)
*Fig.11: Heatmap of Hospitals in Victoria (click on the image for interactive map)*

- The heatmap indicate the density of the hospitals in the region.

## Use cases:
1. **1st use case** - hospital view -  To check for the location of hospitals in Victoria and its Address, And also the type of hospital, whether it is public or private.
  - Public hospitals usually offer cheaper services. And private hospitals are known for more comfort, you can choose based on your preference.
2. **2nd use case** - real estate based -in locality view- If you are looking for a place to stay in a locality with a high number of hospitals you can make a decision based on this map.
  - for example, if you are deciding between Notting hill and ashwood, you might wanna prefer Notting hill as it has 6 hospitals compare to just 1 in Ashwood
3. **3rd use case** - Travel - using heatmap - Suppose you are traveling outside Melbourne and you need to keep hospitals close by. You might prefer the route that takes you through  ballarat or Bendigo over some other place like Castlemaine.

## Conclusion
We can combine all the three features, i.e hospitals, locality and the heatmap in a single map using Featuregroup and control layer.
If you want to check out the **code for combined features visit my github repository [here](https://github.com/nagarajbhat/map-hospitals-victoria)** and open app.py

**To view the app (combining all three features), [click here -Interactive map for hospitals in vistoria](https://www.nagarajbhat.com/app/map-hospitals-victoria.html)**.

Also check out my youtube video for Inteactive maps in victoria - 

{{<youtube uZu8EZr4uYU>}}


