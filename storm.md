# Health and Economic Impact of US Storm Events
datadavidz  
Thursday, April 23, 2015  

## Synopsis
Severe weather events captured in the NOAA Storm database from 1950-2011 have been analyzed to assess the events causing the largest impact on US population health and economy. Tornadoes events were found to responsible for the greatest number of both fatalities and injuries. Flood events were found to be responsible for the greatest economic cost.

## Data Processing

Read the storm data into an R dataframe.  Some data cleanup was performed to adjust for some of the major inconsistencies in the database.  The following changes to the Event Type (EVTYPE) were made:    
1. THUNDERSTORM was standardized to TSTM  
2. WINDS was standardized to WIND  
3. FLOODING was standardized to FLOOD  
  
The units for the damage estimates were then converted from letter designations to numerical multipliers.  
- H or h = 100  
- K or k = 1,000  
- M or m = 1,000,000  
- B or b = 1,000,000,000  
- Everything else = 1  


```r
setwd("c:/Users/David/Documents/GitHub/Storm/")
if (!exists("storm.data")) {
    storm.data <- read.csv(bzfile("repdata-data-StormData.csv.bz2"))

    # Some adjustment of event names to prevent major splitting of the data
    storm.data$EVTYPE <- as.factor(gsub("THUNDERSTORM", "TSTM", storm.data$EVTYPE))
    storm.data$EVTYPE <- as.factor(gsub("WINDS", "WIND", storm.data$EVTYPE))
    storm.data$EVTYPE <- as.factor(gsub("FLOODING", "FLOOD", storm.data$EVTYPE))

    # Conversion of the expense units into multiplication factor
    storm.data$PROPDMGEXP <- gsub("[^HhKkMmBb]", "1", storm.data$PROPDMGEXP)
    storm.data$PROPDMGEXP <- gsub("[Hh]", "100", storm.data$PROPDMGEXP)
    storm.data$PROPDMGEXP <- gsub("[Kk]", "1000", storm.data$PROPDMGEXP)
    storm.data$PROPDMGEXP <- gsub("[Mm]", "1000000", storm.data$PROPDMGEXP)
    storm.data$PROPDMGEXP <- gsub("[Bb]", "1000000000", storm.data$PROPDMGEXP)
    storm.data$PROPDMGEXP[storm.data$PROPDMGEXP == ""] <- "1"
    storm.data$PROPDMGEXP <- as.numeric(storm.data$PROPDMGEXP)

    # Conversion of the expense units into multiplication factor
    storm.data$CROPDMGEXP <- gsub("[^HhKkMmBb]", "1", storm.data$CROPDMGEXP)
    storm.data$CROPDMGEXP <- gsub("[Hh]", "100", storm.data$CROPDMGEXP)
    storm.data$CROPDMGEXP <- gsub("[Kk]", "1000", storm.data$CROPDMGEXP)
    storm.data$CROPDMGEXP <- gsub("[Mm]", "1000000", storm.data$CROPDMGEXP)
    storm.data$CROPDMGEXP <- gsub("[Bb]", "1000000000", storm.data$CROPDMGEXP)
    storm.data$CROPDMGEXP[storm.data$CROPDMGEXP == ""] <- "1"
    storm.data$CROPDMGEXP <- as.numeric(storm.data$CROPDMGEXP)
}
```

## Results
### Population Health Impact  
The top five event types in terms of total number of fatalities plus injuries were determined using the following code.

```r
library(dplyr, warn.conflicts = FALSE, quietly=TRUE)
library(reshape2)
#Calculation of the total fatalities plus injuries for each event
health <- storm.data %>% 
          select(EVTYPE, FATALITIES, INJURIES) %>% 
          group_by(EVTYPE) %>% 
          summarize(EVENTS = n(), FATALITIES = sum(FATALITIES), INJURIES = sum(INJURIES)) %>%
          mutate(TOTINJ = FATALITIES + INJURIES) %>%
          arrange(desc(TOTINJ))
# Reorder the factors according to arrangement for ggplot
health.topfive <- health[1:5,]
health.topfive$EVTYPE <- as.character(health.topfive$EVTYPE)
health.topfive$EVTYPE <- factor(health.topfive$EVTYPE, levels=unique(health.topfive$EVTYPE))
health.topfive <- melt(health.topfive, id = c("EVTYPE", "EVENTS"), measure.vars = c("FATALITIES","INJURIES"))
```
  
The R code for constructing the plot for health impact is presented below.

```r
library(ggplot2)
ggplot(data=health.topfive, aes(x=EVTYPE, y=value, fill=variable)) + 
        geom_bar(stat="identity") +
        ggtitle("Storm Events with Largest Health Impact") +
        xlab("Event Type") +
        ylab("# of Injuries & Fatalities") +
        theme(legend.position=c(0.9,0.8), legend.title=element_blank())
```

![](storm_files/figure-html/unnamed-chunk-3-1.png) 
  
**Figure 1** above shows the top five weather events most impacting population health.  Tornadoes were identified as, by far, causing the greatest number of injuries and fatalities.
  
### Economic Impact
The top five event types in terms of total property and crop damage expense were calculated in $US billions.

```r
# Calculation of the total damage costs in billions
economy <- storm.data %>% 
          mutate(PROPDMG = PROPDMG * PROPDMGEXP / 1e9, CROPDMG = CROPDMG * CROPDMGEXP / 1e9) %>%
          group_by(EVTYPE) %>% 
          summarize(EVENTS = n(), PROPDMG = sum(PROPDMG), CROPDMG = sum(CROPDMG)) %>% 
          mutate(TOTDMG = PROPDMG + CROPDMG) %>%
          arrange(desc(TOTDMG))

# Reorder the factors according to arrangement for ggplot
economy.topfive <- economy[1:5,]
economy.topfive$EVTYPE <- as.character(economy.topfive$EVTYPE)
economy.topfive$EVTYPE <- factor(economy.topfive$EVTYPE, levels=unique(economy.topfive$EVTYPE))
economy.topfive <- melt(economy.topfive, id = c("EVTYPE", "EVENTS"), measure.vars = c("PROPDMG","CROPDMG"))
```

The R code for constructing the plot for economic impact is presented below.

```r
ggplot(data=economy.topfive, aes(x=EVTYPE,y=value, fill=variable)) + 
        geom_bar(stat="identity") +
        ggtitle("Storm Events with Largest Economic Impact") +
        xlab("Event Type") +
        ylab("Total Cost ($Billion USD)") +
        theme(legend.position=c(0.9,0.8), legend.title=element_blank()) +
        scale_fill_discrete(labels=c("Property\nDamage", "Crop\nDamage"))
```

![](storm_files/figure-html/unnamed-chunk-5-1.png) 
  
**Figure 2** above shows the top five weather events in terms of economic impact.  Flood and Hurricanes were identified as the most costly events.
