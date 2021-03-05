# Readme
### The GH file in this project has 2 GH python components inside.
![enter image description here](https://github.com/snjsomnath/GH-OSM/blob/master/img_01.png?raw=true)
 1. The first one allows the user to search for a building coordinate by
    searching for an address 
    
 2. The second one allows the user to convert OSM lat and lon degree
	  decimal values to Rhino Points.

### Read  the following readme files for more instructions

    01 Downloading OSM File.md
    02 Translate Lat Lon to Points.md
# Readme
![enter image description here](https://github.com/snjsomnath/GH-OSM/blob/master/img_01.png?raw=true)
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


# OSM to Points
![enter image description here](https://github.com/snjsomnath/GH-OSM/blob/master/img_01.png?raw=true)
This example is to demonstrate how decimal degree coordinates can be translated to cartesian points

## Load Libraries

    import rhinoscriptsyntax as rs
    import xml.etree.ElementTree as ET
    import ghpythonlib.treehelpers as th
    import math

##	Setup Constants
    R = 6371000

## Parse Lat Lon
    lat = []
    lon = []
    tree = ET.parse(filePath)
    root = tree.getroot()
    for element in root.findall('node'):
        lat.append(float(element.get('lat')))
        lon.append(float(element.get('lon')))

## Parse Bounds

    for element in root.findall('bounds'):
        minlat = float(element.get('minlat'))
        minlon = float(element.get('minlon'))
        maxlat = float(element.get('maxlat'))
        maxlon = float(element.get('maxlon'))

## Find midpoint (Centre of Bounding Box)

    midlat = (minlat+maxlat)/2
    midlon = (minlon+maxlon)/2

# Calculate distance per degree
## Math behind the code
![Cut-away of the globe to show latitude and longiture](https://assets-news-bcdn.dailyhunt.in/cmd/resize/400x400_80/fetchdata16/images/5d/6e/85/5d6e85340a5a95f1a0303a3995b193ec75119de20db49fe988895ae524205739.jpg)

> The Longitude distance per degree is along the x-axis remains constant around the circumference of the earth
> The Latitude distance per degree however, varies. It is largest by the equator and shorter towards the pole since the Latitude lines are parallel to oneanother.
### Y Distance
> (2 * pi * R) Radian Distance = 360 degrees

> Y Distance per degree = (pi * R) / 180 ***(Where R is the radius of the earth in meters - 6371000m)***  
### X Distance
> X Distance per degree = cosine(Latitude) * Y Distance per degree

    ydistperdegree = (math.pi*R)/180
    xdistperdegree = ((math.cos(midlat*(math.pi/180))*R)*(math.pi/180))

## Calculate x and y component of points

    x_comp = []
    y_comp = []
    for i in lon:
        x_comp.append((i-minlon)*xdistperdegree)
    for i in lat:
        y_comp.append((i-minlat)*ydistperdegree)
    Points = []
    for i in range(len(x_comp)):
        Points.append(rs.CreatePoint(x_comp[i],y_comp[i],0))

## Create boundary rectangle

    plane = rs.WorldXYPlane()
    Boundary = rs.AddRectangle(plane,((maxlon-minlon)*xdistperdegree),((maxlat-minlat)*ydistperdegree))

