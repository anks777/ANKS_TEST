# Details of test performed with an application to agriculture and the nitrogen cycle 

Setup information: the work is done under windows 10 (64bit) system, using RStudio version 4.2.0, QGIS 3.22.13, and Microsoft Excel. Below are the stepwise description and codes for each task provided.

-----------------------------------------------------------------------------------------------------------------

# task1: Using SPAM raster data, produce a new raster at the same resolution,.....
#Loading the raster package
library(raster)

#Loading the harvested area raster and yield raster
harvested_area <- raster("data_IEM/SPAM_2005_v3.2/SPAM2005V3r2_global_H_TA_WHEA_A.tif")
yield_raster <- raster("data_IEM/SPAM_2005_v3.2/SPAM2005V3r2_global_Y_TA_WHEA_A.tif")

#Multiplying harvested area and yield to get production volume in kg
production_volume_kg <- harvested_area * yield_raster

#Converting production volume from kg to million tonnes
production_volume_Mt <- production_volume_kg / 1e9  # 1 tonne = 1,000,000 grams = 1e6 grams

#Saving the new raster as a GeoTIFF file
writeRaster(production_volume_Mt, "Wheat_production_volume.tif", format = "GTiff", overwrite = TRUE)

-----------------------------------------------------------------------------------------------------------------

# task2: Using the newly created raster and the GAUL shapefile .....
#Loading required library
install.packages("sf")
library(sf)

#Reading the shapefile
shapefile <- st_read("data_IEM/GAUL/g2015_2005_2.shp")

#Checking the structure of the shapefile
str(shapefile)

#Fixing invalid geometries
clean_shapefile <- st_make_valid(shapefile)

#Dissolving polygons based on a specific field: ADM0_NAME
dissolved_shapefile <- clean_shapefile %>%
  group_by(ADM0_NAME) %>%
  summarise() %>%
  st_union()

# Error in processing GAUL shapefile
Error in wk_handle.wk_wkb(wkb, s2_geography_writer(oriented = oriented,  : Loop 0 is not valid: Edge 40 crosses edge 42

#Alternative attempt to correct dissolve the GAUL shapefile: 

due to difficulties in further processing, QGIS 3.22.13 software is used for correcting the shapefile. 
The steps in QGIS:
- adding the vector layer (the given shapefile)
- using ‘Fix Geometries’ function from the data processing tools
- Saving the new shape file with fixed geometries
- adding vector layer (fixed geometry shapefile)
- Dissolving subnational boundaries (38189 polygons) to national boundaries (273 polygons) using ‘vector geoprocessing tool with Dissolve option’
- The resultant shapefile ‘dissolved.shp’ is extracted in result folder and is used for the steps ahead.

# Using the updated ‘dissolved.shp’ for further analysis

#Loading the raster file
raster_obj <- raster("Wheat_production_volume.tif")

#Loading the vector file
vector_obj <- shapefile("results/dissolved/dissolved.shp")

#Performing zonal statistics (sum) for the raster values within each country polygon
extracted_values <- extract(raster_obj, vector_obj, fun=sum, na.rm=TRUE)

#Error: the previous code could not processed during many hours. 

#Alternative attempt: using other country-level shapefile
For this, the ‘World_Countries.shp’ file was added to the result folder

#Loading the new vector file
vector_obj_new <- shapefile("results/World_Countries/World_Countries.shp")

#Performing zonal statistics (sum) for the raster values within each country polygon
extracted_values <- extract(raster_obj, vector_obj_new, fun=sum, na.rm=TRUE)

#Creating a data frame with the extracted values and country names
result <- data.frame(Country = vector_obj_new$COUNTRY, SumRasterValues = extracted_values)

#Printing the result
print(result)

##Writing the result as a CSV file
write.csv(result, file = "countrylevel_production_result.csv", row.names = FALSE)

-----------------------------------------------------------------------------------------------------------------


# task 3: Using the raster of wheat production generated in question 1, and assuming that 2%.......

#Performing the multiplication to create a new raster (2% of the production volume)
N_output_raster <- raster_obj * 0.02

#Plotting the new raster
plot(N_output_raster)

#Saving the new raster as a GeoTIFF file
writeRaster(N_output_raster, filename = "N_output_raster.tif", format = "GTiff", options = "COMPRESS=LZW")

-----------------------------------------------------------------------------------------------------------------

# task 4: Using the dataset of country-level nitrogen use efficiency (NUE).....

# 4a. estimate for the 10 biggest wheat producers....

- these steps were performed under Microsoft Excel using the CSV file produced in second task.
- Top ten countries in terms of production were extracted with their N output values.
- the values of NUE from the given data were added for corresponding countries. 
- Dividing NUE by N output provided N input data
- Substracting N output from N input provided N losses (surplus)
- the results are saved as CSV file named ’10 countries’ in the result folder.

# 4b. visualize the N outputs and losses for....

- a bar graph was created using these stats to visualize country-level N output and N loss patterns.
- the chart is available as PDF document named ‘chart’ in the result folder.

# 4c. explain in 2-3 sentences the main patterns of N losses....

There is a relationship between production volume and nitrogen losses, with higher production often associated with higher losses. Additionally, NUE varies widely across countries, indicating differences in the efficiency of nitrogen use in agriculture, with room for improvement in several nations, particularly those with significant nitrogen losses like China and India.

-----------------------------------------------------------------------------------------------------------------

# 5. Explain in 2-3 sentences how an analysis like the one......

This analysis presents scope to manipulating and synergising spatial and non-spatial information to derive meaningful indictor, depicting a critical environmental problem. The analyses also enabled to check environmental impacts of nitrogen use related to agricultural land use. These aspects of the analyses translate, to some extent, to the models within BNR’s modelling suite, specifically to the GLOBIOM model.

-----------------------------------------------------------------------------------------------------------------

# 6. Please report any issues you encountered or....

As described in task 2, the shapefile from GAUL couldn’t be processed properly in the RStudio. Hence, an alternative shapefile is used.

-----------------------------------------------------------------------------------------------------------------
# Description of result files in the ‘results’ folder

- dissolved.shp: under the shapefiles folder ‘dissolved’ for the dissolved version of GAUL shapefile, created in QGIS
- World_Countries.shp : alternative shapefile under the shapefiles folder ‘World_Countries’
- Map1: a visualization map from task1 created with QGIS
- Map2: a visualization map from task3 created with QGIS
- countrylevel_production_result.csv: CSV file of country level production from task2
- 10 countries.csv: states related to top ten countries from task4a
- chart: N outputs and losses for 10 countries in 1 summary figure from task 4b
- N_output_raster.tif: a raster of N output in harvested wheat yield from task 3
- Wheat_production_volume.tif: a raster of wheat production from task 1
