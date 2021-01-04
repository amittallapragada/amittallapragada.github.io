---
layout: post
title:  "I reverse engineered Panda Express's Internal API to map out every store that has Eggplant Tofu"
date:   2021-01-03 20:37:09 -0800
categories: docker jupyter notebook python apis
author:  Amit Tallapragada
thumbnail: /assets/2021-01-03-egg-plant-tofu-finder/banner.png
quicklook: "Sending SSE with Python FastAPI"

---
![Banner](/assets/2021-01-03-egg-plant-tofu-finder/banner.png)

TLDR: I made a map with every Panda Express location in the USA that serves Eggplant Tofu [here](https://www.google.com/maps/d/edit?mid=1jxiifURZifURWM6uczPJyNgcwgzcrGw_&usp=sharing)

...

# I reverse engineered Panda Express's Internal API to map out every location that has Eggplant Tofu

If you are vegetarian you already know that, Panda Express is a magical place. My go to meal - a Panda Bowl with Eggplant Tofu has been my STAPLE item for the past 10 years. However, after going to college at Arizona, I learned that not all Pandas are created equally. Some locations DONT HAVE EGGPLANT TOFU. Although this travesty may never be solved in our lifetimes, if you are curious and want to know which ones do serve them- follow along.

### General Idea
In order to do this, we will be "ordering" food from every single panda express in the United States. When you order food from the panda express website, their backend servers return every single menu item that the particular store you are ordering from offers. We will simply check for eggplant tofu in this list. However, in order to do this, we are gonna need a list of every Panda Express in the USA.

### Step 1: Get all list of all the Panda Express locations in the USA
After doing some digging, I found that Panda has an API for this. However, the API returns every single piece of metadata you could want on each of their locations. Because of this, the request takes a long time. I would recommend saving the response from the request to a file so you dont have to keep making this request.
Here is the request you need:

```python
import json
import requests

resp = requests.get(url="https://nomnom-prod-api.pandaexpress.com/restaurants/111469/menu?nomnom=add-restaurant-to-menu")
data = resp.json()
#lets save the data in case we need it again later
with open("pandaLocations.json", "w") as f:
    json.dump(data,f)

#the data dictionary contains various keys we dont care about. We just want the key that contains all the restuarant data
restaurants = data['restaurants']

```


### Lets filter this data
the restaurants key contains all the data you could ask for about any panda express location. The only things we really need to map whether a location has eggplant tofu is the id, latitude, and longitude keys shown below. The 'id' parameter is a unique identifier the panda express internal API uses to refer to each of its stores. We will use this value to hit another one of their APIs which will give us the menu for a specific store. 


```python
restaurants[0].keys()
```




    dict_keys(['acceptsordersbeforeopening', 'acceptsordersuntilclosing', 'advanceonly', 'advanceorderdays', 'allowhandoffchoiceatmanualfire', 'attributes', 'availabilitymessage', 'brand', 'calendars', 'candeliver', 'canpickup', 'city', 'contextualpricing', 'country', 'crossstreet', 'customerfacingmessage', 'customfields', 'deliveryarea', 'deliveryfee', 'deliveryfeetiers', 'distance', 'extref', 'hasolopass', 'id', 'isavailable', 'iscurrentlyopen', 'labels', 'latitude', 'longitude', 'maximumpayinstoreorder', 'metadata', 'minimumdeliveryorder', 'minimumpickuporder', 'mobileurl', 'name', 'productrecipientnamelabel', 'requiresphonenumber', 'showcalories', 'slug', 'specialinstructionsmaxlength', 'state', 'storename', 'streetaddress', 'suggestedtippercentage', 'supportedcardtypes', 'supportedcountries', 'supportedtimemodes', 'supportsbaskettransfers', 'supportscoupons', 'supportscurbside', 'supportsdinein', 'supportsdispatch', 'supportsdrivethru', 'supportsfeedback', 'supportsgrouporders', 'supportsguestordering', 'supportsloyalty', 'supportsmanualfire', 'supportsnationalmenu', 'supportsonlineordering', 'supportsproductrecipientnames', 'supportsspecialinstructions', 'supportssplitpayments', 'supportstip', 'telephone', 'url', 'utcoffset', 'zip'])



#### Looping through the data and filtering for the data we want


```python
eggPlantData = {}
for restaurant in restaurants:
    eggPlantData[restaurant['id']] = {"lat":restaurant['latitude'], 
                                      "lng":restaurant['longitude'], 
                                      "name":restaurant['name'],
                                      "state":restaurant['state']
                                    }
```

Ok awesome. Now our 'eggPlantData' has all the data we need on each restaurant. It is a dictionary where the key is the store id and the value is the store's geocoordinates and street address. Lets take a look at a particular stores entry in 'eggPlantData'.

```python
eggPlantData[111469]
```




    {'lat': 37.331691,
     'lng': -121.810975,
     'name': 'E Capitol Expwy & Tully',
     'state': 'CA'}


So our list of Pandas Express stores across America is now built. Now lets figure out how to get each store's menu items. 

### Step 2: Checking each stores menu items
Panda Express has another API that will tell us a particular stores menu items. The only parameter it takes is the store 'id' value. Luckily that value is the key in our 'eggPlantData' dictionary. Lets take a look at an example request:


```python
store_id = 111469
resp = requests.get(url=f"https://nomnom-prod-api.pandaexpress.com/restaurants/{store_id}/menu?nomnom=add-restaurant-to-menu")
store_data = resp.json()
print(store_data)
```

    {'categories': [{'description': '1 Side & 1 Entree', 'id': 39038, 'images': [{'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'mobile-app', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'mobile-webapp-menu', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'mobile-webapp-customize', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=716&h=474&fit=crop&fm=png32&s=5c543defe38946e36a8694d0b149fda4', 'groupname': 'desktop-menu', 'isdefault': False, 'url': None}], 'metadata': [{'key': 'layout', 'value': 'show-products'}], 'name': 'Bowl', 'products': [{'availability': {'always': True, 'description': None, 'enddate': None, 'now': True, 'startdate': None}, 'basecalories': None, 'caloriesseparator': None, 'chainproductid': 262728, 'cost': 7, 'costoverridelabel': None, 'description': 'Any 1 Side & 1 Entree', 'displayid': None, 'endhour': 24, 'id': 24144449, 'imagefilename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'images': [{'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=1200&h=800&fit=fill&fm=png32&bg=transparent&s=1d85a6644c57f3f16416b50d525b6956', 'groupname': 'marketplace-product', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'mobile-app', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'mobile-app-large', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=524&h=350&fit=crop&fm=png32&s=684050f6bee064071c0a39c31bafd6f6', 'groupname': 'mobile-webapp-menu', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'mobile-webapp-customize', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'desktop-menu', 'isdefault': False, 'url': None}, {'description': None, 'filename': '72/7288570f72a54140a41afdcfbd0e8980.png?auto=format%2Ccompress&q=60&cs=tinysrgb&w=810&h=540&fit=crop&fm=png32&s=8aff41fd52f99c8e28ffb83d51c3c685', 'groupname': 'desktop-customize', 'isdefault': False, 'url': None}], 'maxcalories': None, 'maximumbasketquantity': '', 'maximumquantity': None, 'menuitemlabels': [], 'metadata': [{'key': 'layout', 'value': 'show-products'}], 'minimumbasketquantity': '', 'minimumquantity': '1', 'name': 'Bowl', 'quantityincrement': '1', 'shortdescription': '', 'starthour': 0, 'unavailablehandoffmodes': ['dinein'], 'slug': 'bowl'}], 'slug': 'bowl'}, {'description': '1 Side & 2 Entrees', 'id': 39030, .....}


As you can see we get back a lot of unecessary data. I had to cut off the amount of the response I displayed in order for my blog to be able to display it. The json is tricky to traverse. Essentially, has a key called "Categories". This key contains information on the different types of food formats Panda has to offer. These are things like their 2 entree special, the Panda Bowl, or Individual Entrees & Sides. In our case, we are interested with only Eggplant Tofu. Thus we will be searching the Categories key for an entry called 'Individual Entrees & Sides'. If we find it we will traverse another nested dictionary within it called 'Products'. This dictionary will contain all the individual items a particular store carries. Here we can finally search for an entry called "Eggplant Tofu". Lets see this in action for a single store's data:  


```python
categories = store_data['categories']
individual_dishes = None
for cat in categories:
    if cat['description'] == "Individual Entrees & Sides":
        individual_dishes = cat['products']
        break
for dish in individual_dishes:
    if dish['name'] == "Eggplant Tofu":
        print("I HAVE EGGPLANT TOFU!!!")
        break
```

    I HAVE EGGPLANT TOFU!!! 


Perfect. The code snippet above will tell you if a particular store has eggplant tofu. 
Now lets run it for every single store in america.

### Step 3: Running it for Every Panda Express in America

```python
for store_id, data in eggPlantData.items():
    print(store_id)
    resp = requests.get(url=f"https://nomnom-prod-api.pandaexpress.com/restaurants/{store_id}/menu?nomnom=add-restaurant-to-menu")
    store_data = resp.json()
    #get categories
    categories = store_data['categories']
    individual_dishes = None
    eggPlantData[store_id]['has_eggplant'] = False
    for cat in categories:
        if cat['description'] == "Individual Entrees & Sides":
            individual_dishes = cat['products']
            break
    if individual_dishes == None:
        print(f'no individual dishes for: {store_id}')
    else:
        for dish in individual_dishes:
            if dish['name'] == "Eggplant Tofu":
                eggPlantData['has_eggplant'] = True
                break
        
with open("eggPlantFinder.json",'w') as f:
    json.dump(restaurant,f)
```

Let me explain this snippet. We are looping through our eggPlantData dictionary and calling the store api for each store in the dict. We then check if that particular store has eggplant tofu or not. We store this in our eggPlantData dictionary as another key called "has_eggplant". This code takes a long time to run so I saved it to a file. I'll make it available on my github as well. 

Perfect! We could just stop here. We have a list of every panda express along with its coordinates that have eggplant tofu. But it wouldn't be a complete project until we map it...

### Step 4: Lets Map It

I experimented with a bunch of different ways of mapping this project. However, I reasoned that the easiest way to do this was with google maps. Its easily shareable, simple to add markers too, and easy to embed on websites. So to do this, lets first format our data. For this I'm probably using the most overkill approach of exporting the 'eggPlantData' dictionary to a csv by using the pandas library.

```python
import pandas as pd
geoCoordInfo = []
for store, data in eggPlantData.items():
    geoCoordInfo.append(data)

df = pd.DataFrame(data=geoCoordInfo)
df.to_csv("panda_eggplant.csv")
```
But its also super clean :P 

Now its just a matter of importing our data into google maps and selecting a custom logo for locations with/without eggplant tofu. I won't cover this part in detail as google has a great tutorial on doing [this](https://www.google.com/earth/outreach/learn/visualize-your-data-on-a-custom-map-using-google-my-maps/#prerequisites-)

### Step 5: We made it

<iframe src="https://www.google.com/maps/d/embed?mid=1jxiifURZifURWM6uczPJyNgcwgzcrGw_" width="740" height="480"></iframe>

Here it is. Our glorious map to freedom. I hope you guys enjoyed this journey as much as I have. Please let me know if you have any questions or comments! In the meantime, im gonna go eat some eggplant tofu...

<p align="center">
<img src="/assets/2021-01-03-egg-plant-tofu-finder/eggplantTofu.jpg" width="400"/>
</p>

 
<div>Icons made by <a href="https://www.flaticon.com/authors/freepik" title="Freepik">Freepik</a> from <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a></div>
