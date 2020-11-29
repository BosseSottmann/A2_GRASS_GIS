## Answers to the GRASS assignment

### 1. Create new location in GRASS GIS
Open GRASS, browse to folder where one wants to have the new location. Name it, and during the next steps, watch out to alter the coordinate system and projection to the ones of the given TIFF. Do this by locating this during the file explorer during the setup. Also: be sure to directly import the TIFF as one needs it later on.
Click create and move on!

### 2. Import data

#### 2.1 Motorways

v.import motorways.shp

Is projected 'on the fly'

##### 2.2 Administrative Districts of Baden-Württemberg

v.in.ogr input=gadm28_adm2_germany.shp layer=gadm28_adm2_germany output=BW_adm2_unproj where=ID_1=1 snap=1e-009 location=side_loc

v.proj location=side_loc input=BW_adm2_unproj output=BW_adm2

split up into two commands as the command wrapper with v.import didn't work out. This one does. Disadvantage: Creation of an extra location.


##### 2.3 Global Human Settlement Layer

Already done during the creation.

### 3. Calculate the total population of the districts

The goal of this section is to calculate the total population of each district.

##### 3.1 Set the region

g.region vector=BW_adm2 res=250

Sets the region to the extent of the district layer and puts the resolution of the region to 250 by 250 meters. Even though, the exact resolution is nsres=249.89707939 and ewres=249.95341164.

##### 3.2 Rasterize the districts

v.to.rast input=BW_adm2 output=BW_adm2_rast use=attr attribute_column=OBJECTID

The attribute_column together with the use command specify the input for the cells

##### 3.3 Calculate the population of each district

r.stats.zonal base=BW_adm2_rast cover=GHS_POP_E2015_GLOBE_R2019A_54009_250_V1_0_18_3 method=sum output=BW_adm2_pop_sum

aggregation method=sum to get a sum of all values in the corresponding district

#### 3.4. Evaluate the population estimate

Official data from [Statistisches Landesamt Baden-Württemberg](https://www.statistik-bw.de/BevoelkGebiet/Bevoelk_I_D_A_vj.csv)

Landkreis Rhein-Neckar-Kreis official: 546.914
Landkreis Rhein-Neckar-Kreis calculated: 534.574

Landkreis Main-Tauber-Kreis official: 132388
Landkreis Main-Tauber-Kreis	calculated: 130421

Based on these two, the calculated estimates are relatively usable. For a complete evaluation, each district would need to be compared side by side and then statistically analysed.


### 4. Calculate total population living within 1km of motorways

`g.region vector=motorways`

`v.buffer input=motorways output=motorways_buffered type=line distance=1000`

``v.to.rast input=motorways_buffered output=motorways_buffered_rast use=cat ``

``v.db.addtable motorways_buffered``
had to be added since the datatable of the buffer was not connected to its database. This one took some time to figure out.. (db.connect etc. didn't work..)

``r.stats.zonal base=motorways_buffered_rast cover=GHS_POP_E2015_GLOBE_R2019A_54009_250_V1_0_18_3 method=sum output=motorways_pop_sum``

`r.stats`:  
100%
299503.5625-299503.5625
*


### 5. Convert the script to Python

1. Convert the workflow described in section 4 to a Python script.

2. Use a for loop to calculate the population living close to motorways using different buffer distances: 250, 500, 1000, 2500 and 5000 meters.


#### References

[GRASS GIS 7.8 Manual](https://grass.osgeo.org/grass78/manuals/index.html/)  
