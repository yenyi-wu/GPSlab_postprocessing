# GPSlab_postprocessing (matlab)

## Introduction

GPS lab[^1] is a free open data source platform built based on the National Science Council project “Establishing an Automated GPS Processing and Analysis System to Provide an On-line Service for the Earth Sciences Research” in Taiwan. The platform offers GPS time series data collected from 539 GPS stations all over Taiwan with the time period from 1993 to 2019 and an hourly interval. The RINEX files collected from GPS are transformed to Bernese specific format, and then processed with Bernese software. Therefore, the data downloaded from GPS lab are end products and could be directly applied.

The dates recorded in GPS lab are in floating points. For example, 2000.38388. By this, you only know the data was recorded in year 2000 but do not know the date. Therefore, this code deals with two main problems: 
1. convert the date format; 
2. select specific dates of data you need.

## Relationship between the recorded format (D) and day of year (DOY)
- D = (DOY-0.5)/366
- DOY = D*366+0.5

Therefore, for the record "2000.38388", year is 2000, D is 0.38388, and DOY is 141 according to the relationship given above.
One can check the [Day-Of-Year Calendar](https://www.esrl.noaa.gov/gmd/grad/neubrew/Calendar.jsp?view=DOY&year=2019&col=4) to look up the exact date of the year. In this example, 2000.38388 is May 20th in year 2000. 

## Code explanation
The code is written in matlab.  
Before coding, I firstly looked up the DOY of the specific dates I need, and arranged the information into a new file called "Dates". The following figure displays the contents of my "Dates" file. This file is needed for the subsequent program.   
![image](https://user-images.githubusercontent.com/72783662/138560687-4681b7bb-6313-4af4-ad05-60cffb3ec884.png)

### 1. Read the "Dates" file
Stores the day of year information in the variable **DOY**. Then convert them to the recorded format, and save the values as **D**.
```
cd G:\2.GPS\2.FilterDates
fid = fopen('Dates.txt','r');
Dates = textscan(fid,'%s %d', 'MultipleDelimsAsOne',1);
d = cell2mat(Dates{1});

year = [];
for i = 1:length(d)
    year = [year; d(i,1:4)];
end
year = str2num(year);

DOY = Dates{2};
D = (double(DOY)-0.5)./366-0.0001;
DOY_2017 = [double(year(1:4)) double(DOY(1:4)) D(1:4)];
DOY_2018 = [double(year(5:20)) double(DOY(5:20)) D(5:20)];
DOY_2019 = [double(year(21:38)) double(DOY(21:38)) D(21:38)];
fclose(fid);
```

### 2. Read GPS data and filter the data according to the year
First of all, the GPS data are stored as .COR files.  
Secondly, collect data between 2017 and 2019.
Thirdly, compare the recorded date with with **D**. If the difference lies within a certain range, consider the dates match.
Fourthly, merge the results between 2017 and 2019, and store the output as a table. 
```
cd G:\2.GPS\1.Data
files = dir('*.COR');

for i=1:length(files)
    fid(i) = fopen(files(i).name);
    formatSpec =  [repmat(' %s',1,1) repmat(' %f',1,7)];
    files(i).values = textscan(fid(i), formatSpec,'delimiter',',','MultipleDelimsAsOne',1);
    name = files(i).name;
    values = cell2mat(files(i).values{1});
    
    y = []; date = []; Lat = []; Lon = []; Hgt = [];
    
    for l = 1:length(values)
        y = [y; values(l,1:4)];
        date = [date; values(l,5:10)];
        Lon = [Lon; values(l,27:40)];
        Lat = [Lat; values(l,12:24)];
        Hgt = [Hgt; values(l,44:50)];
    end

    Ori_2017 = [];Ori_2018 = [];Ori_2019 = [];
    y = str2num(y); date = str2num(date); Lon = str2num(Lon); Lat = str2num(Lat); Hgt = str2num(Hgt);

    for j = 1:length(y)  
        if y(j) == 2017
            Ori_2017 = [Ori_2017; double(y(j)) date(j) Lon(j) Lat(j) Hgt(j)];
        elseif y(j) == 2018
            Ori_2018 = [Ori_2018; double(y(j)) date(j) Lon(j) Lat(j) Hgt(j)];
        elseif y(j) == 2019
            Ori_2019 = [Ori_2019; double(y(j)) date(j) Lon(j) Lat(j) Hgt(j)];
        end
    end
    
    pick_2017 = [];
    if isempty(Ori_2017) == 0
        for m = 1:length(DOY_2017)
            s = abs(Ori_2017(:,2)-DOY_2017(m,3));
            if min(s) <= 0.006;
                ind = find(s == min(s));
                pick_2017 = [pick_2017; DOY_2017(m,1:3) Lon(ind) Lat(ind) Hgt(ind)];
            else
                fprintf('%s: 2017.%d, smallest gap is %f days\n', name, DOY_2017(m,2),round(min(s)/0.0027))
            end
        end
    end
    
    pick_2018 = [];
    if isempty(Ori_2018) == 0
        for m = 1:length(DOY_2018)
            s = abs(Ori_2018(:,2)-DOY_2018(m,3));
            if min(s) <= 0.006;
                ind = find(s == min(s));
                pick_2018 = [pick_2018; DOY_2018(m,1:3) Lon(ind) Lat(ind) Hgt(ind)];
            else
                fprintf('%s: 2018.%d, smallest gap is %f days\n', name, DOY_2018(m,2),round(min(s)/0.0027))
            end
        end
    end
        
    pick_2019 = [];
    if isempty(Ori_2019) == 0
        for m = 1:length(DOY_2019)
            s = abs(Ori_2019(:,2)-DOY_2019(m,3));
            if min(s) <= 0.006;
                ind = find(s == min(s));
                pick_2019 = [pick_2019; DOY_2019(m,1:3) Lon(ind) Lat(ind) Hgt(ind)];
            else
                fprintf('%s: 2019.%d, smallest gap is %f days\n', name, DOY_2019(m,2),round(min(s)/0.0027))
            end
        end
    end
        
    pick_all = [pick_2017; pick_2018;pick_2019];
    if isempty(pick_all) == 0
        T = splitvars(table(pick_all));
        T.Properties.VariableNames = {'Year','DOY','date','Lon.(X)','Lat.(Y)','Hgt'};
        my_directory = 'E:\1.GPS\1.filter_38_dates';
        T_name = [my_directory filesep  name(1:4) '.txt'];
        writetable(T,T_name);
    end
end
```


[^1]: The platform is currently not in service, so the data is not accessible anymore. 
