# GPSlab_postprocessing
Postprocessing for GPS data downloaded from GPS lab. 

The dates recorded in GPS lab are in floating points. For example, 2000.38388. By this, you only know the data was recorded in year 2000 but do not know the specific recording date. Therefore, this code deals with two main problems: 
(1) convert the date format to DOY (day of year); 
(2) select specific dates of data you need.
