## Baltimore Crime Map using R

**Project Description:** Now we have a Baltimore crime timestamp dataset from the Baltimore Police Department.
We would like to know when and where Baltimore had more crimes in a daily timeframe. Thus, we need to plot the crimes on the Baltimore map.

### 1. Load the Baltimore map

For this part, my professor had done all the work so this part of codes credit to Yufeng.

```Rscript
library(maps)
library(maptools)
# download, unzip and read the shape file
url_zip <- 'https://dl.dropboxusercontent.com/s/chyvmlrkkk4jcgb/school_distr.zip'
if(!file.exists('school_distr.zip')) download.file(url_zip, 'school_distr.zip')     # download file as zip
unzip('school_distr.zip')   # unzip in the default folder
schdstr_shp <- readShapePoly('school.shp')  # read shape file
xlim <- schdstr_shp@bbox[1,]
ylim <- schdstr_shp@bbox[2,]
plot(schdstr_shp, axes = T) 
```

### 2. Transform date and time

```Rscript
library(lubridate)
df.raw$CrimeDate1 <- mdy(df.raw$CrimeDate)
df.raw$CrimeTime1 <- hms::as_hms(df.raw$CrimeTime)
```

### 3. Split coordinates into longitude and latitude

```Rscript
df.raw$latitude <- as.numeric(substr(df.raw$Location1,2,9))
df.raw$longitude <- as.numeric(substr(df.raw$Location1,16,25))
```

### 4. Get a subset of "assault" only

This is because we only want to see the assault crime as our professor asked, so as to train us on text data cleaning.

```Rscript
assault = grep( "[Aa][Ss][Ss][Aa][Uu][Ll][Tt]", df.raw$Description)
```

### 5. Divide a day into 4 periods

This is because we want to see the crime differences in different time periods. We divide them into morning (6:00 am to 12:00 pm), afternoon (12:00 pm to 6:00 pm), evening (6:00 pm to 12:00 am) and mid-night (12:00 am to 6:00 am).

```Rscript
df.raw$hour <- hour(df.raw$CrimeTime1)
df.raw$TimePattern <- ifelse(df.raw$hour >=0 & df.raw$hour <6, 1, 
                             ifelse(df.raw$hour >=6 & df.raw$hour <12, 2,
                                    ifelse(df.raw$hour >=12 & df.raw$hour <18, 3, 4)))
```

### 5. Draw the plot
```Rscript
par(mfrow = c(2, 2))                       
library(ggplot2)

main <- c("hour: 0-6","hour: 6-12","hour: 12-18","hour: 18-24")

for(i in 1:4){
  plot(schdstr_shp, axes = T,
       main = main[i])
  points(x = df.raw[df.raw$TimePattern == i,]$longitude,
         y = df.raw[df.raw$TimePattern == i,]$latitude,
         pch = 19, cex = 0.1, col=rgb(1,0,0,0.02))
 ```
 

