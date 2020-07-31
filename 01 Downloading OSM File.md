# Readme

This script downloads an OSM file and saves it to drive from a centre point or a search term

## Input Varaiable

 - search  - [string]              Search query of the location
- midLat  - [float]   (optional)  latitude of the building
- midLon  - [float]   (optional)  longitude of the building
- bbox    - [int]                 Dimension of one side of bounding box for OSM
- path    - [string]              A folder directory to save the OSM File  
- run     - [boolean]             A boolean toggle to download the OSM file (Run once)

# 01 Load Libraries
    import xml.etree.ElementTree as ET
    import ghpythonlib.treehelpers as th
    import math
    import urllib2
    import json

# 02 Generate an API Key from positionstack
www.positionstack.com

    KEY = 'YOUR API KEY HERE'

## Create option to either search or use manual midLat and midLon

    if search:
        print 'Search Tag found' 
        query = search.replace(" ", "%20")
        url = 'http://api.positionstack.com/v1/forward?access_key='+KEY+'&query='+query
        response = urllib2.urlopen(url)
        data = json.load(response)   
        midLon = data['data'][0]['longitude']
        midLat = data['data'][0]['latitude']
    else:
        print 'Search Tag not Found. Enter search term or use midLat and midLon'

## Constants for calculating the bounding box

    R = 6371000

# 03 Bounding Box calculation

#111110 is the distance of 1 degree in meters for the Y direction (Latitude)
#X distance per degree = [cosine(latitude)*Y distance in radians]
#bbox here is the dimension of the boundarybox side

    dY = (bbox/2) / 111110
    dX = dY / math.cos(math.radians(midLat))

## Calculation the extents of the bounding box

    a = str(midLon - dX)
    b = str(midLat - dY)
    c = str(midLon + dX)
    d = str(midLat + dY)
    bbox = a +','+ b + ',' + c + ',' + d

# 04 Requesting OSM file from OverpassAPI

    url = 'https://overpass-api.de/api/map?bbox='+bbox
    if run:
        response = urllib2.urlopen(url)
        filePath = path+'file.osm'
        mydata = response.read()
        file = open(filePath, "w")
        file.write(mydata)
        file.close()
    else:
        filePath = path+'file.osm'
