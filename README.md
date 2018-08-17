# Bay-Area-Bike-Share
An exploration of the Bay Area Bike Share data using SQL and Google BigQuery

## Introduction
In this project, I created and answered business-driven questions using the Bay Area Bike Share Trips Data (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

## Queries

### Part 1: Exploring the dataset

- What's the size of this dataset? (i.e., how many trips)

There are 983648 trips in this dataset.

```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  bigquery-public-data.san_francisco.bikeshare_trips  
```

- What is the earliest start time and latest end time for a trip?

The earliest start time for a trip is 2013-08-29 09:08:00.000 UTC. The latest end time for a trip is 2016-08-31 23:48:00.000 UTC.

```sql
#standardSQL
SELECT
  MAX(end_date)
FROM
  bigquery-public-data.san_francisco.bikeshare_trips' 
```
```sql
#standardSQL
SELECT
  MIN(start_date)
FROM
  bigquery-public-data.san_francisco.bikeshare_trips' 
```

- How many bikes are there?

There are 700 bikes in this dataset.

```sql
#standardSQL
SELECT
  COUNT(DISTINCT bike_number)
FROM
  bigquery-public-data.san_francisco.bikeshare_trips' 
```

- How many trips are in the morning vs in the afternoon?

It depends on how morning and afternoon are defined, and whether we care about when the trip starts or ends. For instance, a trip that starts in the morning and ends in the afternoon could be categorized as morning or afternoon, depending on how that's defined. 

I set the cutoff at noon, and ran the queries for both start and end date. There were 524,359 trips that started after noon and 459,289 trips that started before noon. 

Also, there were 538,637 trips that ended after noon and 445,011 trips that ended before noon. While the numbers are similar, they aren't the same. 

Some thought should be put into how this is defined (and if it matters substantively, or if we just need to pick one or keep running both sets of queries). 

```sql
#standardSQL
SELECT
  COUNT(trip_id)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  EXTRACT (hour
  FROM
    start_date) > 12 
```
```sql
SELECT
  count(trip_id)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  EXTRACT (hour
  FROM
    start_date) <= 12
```
```sql
#standardSQL
SELECT
  count(trip_id)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  EXTRACT (hour
  FROM
    end_date) > 12
```
```sql
#standardSQL
SELECT
  count(trip_id)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  EXTRACT (hour
  FROM
    end_date) <= 12
```

### Part 2: Identifying commuters/regulars versus recreational/tourists

The reason for exploring this aspect is to get a better sense of who Ford GoBike's customers are, and how they use the bikeshare. 

- How many round trips (start and end station are the same) are in this data set? Assuming that riders who use the bikes for round trips could be more likely to use the bikes for recreation/tourism purposes. If a rider is using the bike to commute, however, s(he) is likely to drop the bike off at a different location than where s(he) started. 

Only 32047 rides had the same start and end station. This indicates a likelihood that most rides are actually commuter rides.

```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  start_station_name=end_station_name
```

- How many subscriber rides are in the dataset? This is of interest because, by definition, subscribers are repeat customers. Because they are repeat customers, they are generally easier to retain, and are likely to generate more revenue.

846,839 rides are subscriber rides. This shows a clear majority of rides being done by subscribers. Assuming subscribers are more likely to be commuters and less likely to be taking rides for tourism/recreational purposes, this is consistent with the answer from the previous question.

```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips` 
WHERE
  subscriber_type="Subscriber"
```
- What are the unique zip codes? Even though this data has missing values, there is still useful information here. Specifically, as we are interested in commuters, it's safe to assume commuters are local. However, that needs to be verified. In addition to this, if there are popular zip codes that aren't local (say, from Los Angeles or Seattle), then Ford GoBike would have the option of tailoring special deals to visitors from zip codes most likely to rent the bikes. 

There are 8830 distinct zip codes. This suggests a variety of locations from where people may visit. 

```sql
#standardSQL
SELECT
  COUNT(DISTINCT zip_code)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
```
- How many "nil"'s are listed in the zip code? Since the data set does have some missing values, it is important to see just how much of an issue this is. 

As it turns out, there are only 15,385 'nil' values. Considering the size of this data set, this is only a small fraction of all rides. Because the missing values are only a small fraction (less than 2 percent), I am not considering this to be problematic enough to meaningfully impact this study and integrity of the conclusions and recommendations. 
  
```sql
#standardSQL
SELECT
  COUNT(zip_code)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  zip_code = 'nil'
```

- What are the most popular zip codes for rides? This is simply a continuation of the previous queries. 

Not including the "nil" values, the top 10 most popular zip codes are as follows: 94107 with 106,913 rides, 94105 with 61,232 rides, 94133 with 46,544 rides, 94103 with 38072 rides, 94111 with 33,642 rides, 94102 with 30,222 rides, 94109 with 19,043 rides, 95112 with 15,420 rides, 94158 with 13,673 rides, and finally 94611 with 13,198 rides. 
  
Interestingly, all of these zip codes are local to San Francisco, with the exception of 95112, which is a San Jose zip code and 94611, which is an Oakland zip code. As both of these are local to the Bay Area, it can be assumed that riders from these zip codes are also commuters. They might be daily commuters, or might take the train in and use the bikes to commute once in the city.
  
```sql
#standardSQL
SELECT
  COUNT(*), zip_code
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
GROUP BY
  zip_code
ORDER BY
  count(*) DESC
```

- How many trips from the most popular zip codes are non-subscribers? This is important to check the assumption that the rides from the most popular zip codes are primarily subscribers (and therefore commuters).

For all of these zip codes, only a small fraction are customer rides. From 94017, there are only 14 Customer rides, 1,650 from 94015, 1,302 from 94133, 1,555 from 94013, 1,027 from 94111, 1,505 from 94102, 1,347 from 94019, 1,312 from 95112, 638 from 94158, and 316 from 94611.
  
This is interesting. While these local customer rides represent only a small fraction of local subscriber rides, it may be worth it to target these users. What sort of offer could GoBike make that would result enough customers becoming subscribers so as to be profitable for GoBike? Perhaps there is something useful in between, such as a weekly or monthly membership, that could appeal to these riders.  
 
```sql
#standardSQL
SELECT
  COUNT(*),
  zip_code
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  subscriber_type ='Customer'AND (zip_code='94017'
    OR zip_code='94105'
    OR zip_code='94133'
    OR zip_code='94103'
    OR zip_code='94111'
    OR zip_code='94102'
    OR zip_code='94109'
    OR zip_code='95112'
    OR zip_code='94158'
    OR zip_code='94611')
GROUP BY
  zip_code
ORDER BY
  COUNT(*) DESC
```

- How many trips out there are between 10 minutes and 1 hour in duration? I am using this duration as an approximate commuter trip length.

There are 360,929 trips between 10 minutes and 1 hour in duration. These are good candidates for commuter trips as these approximate commuter trip length. Interestingly, they are roughly a third of the total trips. 
  
```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  duration_sec >= 600
  AND duration_sec <= 3600
```

- How many trips out there are between 1 and 4 hours in duration? These are likely to be recreational and not commuter trips.

There are only 20,804 trips between 1 and 4 hours in duration, which is considerably fewer than the likely commuter trips. 
  
```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips` 
WHERE
  duration_sec >= 3600
  AND duration_sec <= 14400
```

- It is concerning that both the commuter and the recreational trips do not encompass the majority of the trips. Therefore, my next questions is: How many trips are shorter than 10 minutes in duration? These are likely to be user error, and would indicate issues with data integrity. 

As it turns out, there are 594628 rides that are 10 minutes in duration or less. This is almost 2/3 of the data set, which is concerning. 

```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips` 
WHERE
  duration_sec < 600
```

- As a sanity check, my next query cuts the time in half, in case commuters actually do make 5-10 minute commutes using Ford GoBike. 

Fortunately, the number of rides less than 5 minutes is much lower, down to 178692 rides. While this is still problematic for the data set, indicating a high rate of user error, it is much lower than 2/3 of the data set. 

```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips` 
WHERE
  duration_sec < 300
```

This final query re-runs the commuter ride query. However, instead of the duration being from 10 minutes to 1 hour, it is now 5 minutes to 1 hour. 

- How many trips out there are between 5 minutes and 1 hour in duration? 

As expected, the number of commuter rides skyrocketed, to a total of 776865. This is almost 79 percent of total rides, which is a clear, strong majority. 

```sql
#standardSQL
SELECT
  COUNT(*)
FROM
  `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE
  duration_sec >= 300
  AND duration_sec <= 3600
```

### Part 3: Exploring bike demand at various stations

The reason for also exploring this is to see if there is a more efficient way in which Ford GoBike can manage their bike inventory. Are there too many bikes in some locations, and not enough in others? Could this be hurting business?  

- What is the longest duration and what is the shortest duration for a trip? This would give us a range of time when the bikes are in use. While it seems less strategic, when I work with a new dataset, the first thing I do is an EDA to explore a bit. Range is one of those aspects to be examined here. Specifically, it could also give us a sense of possible data integrity issues--for instance, if the lowest duration is only a few seconds, and/or the longest duration is for several days. Rides that only last for a few seconds could be user error, and if there are rides that last several days, the reason for this might be something worth looking into.

The longest ride is 17270400 (translating roughly to a month). I checked on the website, and wasn't able to find a maximum allowed duration for a ride. (https://www.fordgobike.com/how-it-works) From the results of this query, assuming the data is accurate, at least 1 rider kept a bike for a month, which, in my opinion, is gaming the system. This is something that Ford GoBike should consider looking into further. The minimum ride is only 60 seconds, which is consistent with the above queries and the concerns about user error. 

```sql
#standardSQL
SELECT
  MAX(duration_sec)
FROM
  bigquery-public-data.san_francisco.bikeshare_trips
```
```sql
#standardSQL
SELECT
  MIN(duration_sec)
FROM
  bigquery-public-data.san_francisco.bikeshare_trips
```

- What is the maximum number of bikes available at any given time?

The maximum bike availability is 29 bikes. 

```sql
#standardSQL
SELECT
  MAX(bikes_available)
FROM
  `bigquery-public-data.san_francisco.bikeshare_status`
```
  
- What is the maximum number of docks available at any given time?
 
The maximum docking station availability is 35 docking stations. Because this doesn't match bikes available, I'd like to dig in a bit further and see why. Is this a feature, or perhaps an issue with the data?

```sql
#standardSQL
SELECT
  MAX(docks_available)
FROM
  `bigquery-public-data.san_francisco.bikeshare_status` 
```

- How many docks are at each station?

As we can see from the table, the maximum amount is at station 91, with 35 docks available. From the questions below, we know that 91 has low bike availability as well, which indicates a good use of this resource. The minimum number of docking stations are 11 stations, 4, 32, 35, and 37. None of these seem to be particularly popular, which is also an indicator of a good use of the inventory. 

```sql
SELECT
  MAX(dockcount), station_id
FROM
  `bigquery-public-data.san_francisco.bikeshare_stations` 
GROUP BY
  station_id
ORDER BY
  MAX(dockcount) DESC
```
- Which stations have zero dock availability at any given time? 

These stations are 51, 62, 61, 60, 59, and 58. Interestingly, they don't match with the stations that have the lowest bike availability. They have 19, 19, 27, 15, 23, and 19 docking stations available, respectively. None of these stations have as many as the maximum (35). 

```sql
SELECT
  MIN(docks_available), station_id
FROM
  `bigquery-public-data.san_francisco.bikeshare_status` 
GROUP BY
  station_id
ORDER BY
  MIN(docks_available) ASC
```

- Which stations have the most dock availability at any given time? 

The top stations are 83, 24, 22, 21, 14, and 29. Interestingly, they don't match with the stations that have the lowest bike availability. 

```sql
SELECT
  MAX(docks_available), station_id
FROM
  `bigquery-public-data.san_francisco.bikeshare_status` 
GROUP BY
  station_id
ORDER BY
  MIN(docks_available) DESC
```

-Which stations tend to have the most bikes available?

For this query, I didn't take into account time--I just wanted to see which stations had the most bikes available for any given time stamp. It turns out, most of them were in the 1.5 million range. Again, taking into account that these are really by time stamp (so the same bike could be there for hours, and be counted multiple times). This was just to get a sense of any docking stations that tended to have the most bikes available. 
  
However, what was interesting, instead, are the ones with the lowest bike availability. These stations might be the most popular (and, in turn, more bikes could be docked in these stations to increase the capacity that is being used and decrease capacity that is not being used. Below are both SQL queries (they're basically the same, just sorted differently.) As it turns out, the following stations have the least bike availability: station 87 with 167 bikes available, 91 with 8,905 available, 90 with 8,916 available, 89 with 8,945 available, and 88 with 8,965 available. The rest of the stations all have bike availability in the millions. This is consistent with the above discussion on capacity versus demand at any given time and location. 

```sql
#standardSQL
SELECT
  COUNT(bikes_available),
  station_id
FROM
  `bigquery-public-data.san_francisco.bikeshare_status`
GROUP BY
  station_id
ORDER BY
  COUNT(bikes_available) ASC
```

## Report

### Riders:

- The clear majority of rides are local, subscriber rides. To attract more of these repeat customers, the pricing schedule needs to be adjusted. In addition to a yearly subscription, there should be weekly and monthly options as well. Not all potential subscribers would necessarily be willing and able to commit for a year. The weekly rate should be lower than the daily rate, and higher than the monthly rate. Also, the monthly rate should be higher than the yearly rate. This way, it incentivizes commuters to commit for as long as they are comfortable, while still offering flexibility that may fit better with personal preferences. 

- Considering that each minute a bike is not in use is a lost opportunity, more should be done to also bring in recreational riders. In addition to the weekly pass, the daily pass needs to be changed. Instead of unlimited 30 minute rides (which isn't even practical for all commuters), it should be simply unlimited use for a day. While most rides are short in duration, this may encourage more longer duration rides at times when the bikes would otherwise be idle. 

	- In addition to this, there could be discounts on single rides for mid-day weekdays and weekends. The purpose of this would be to encourage recreational riding when the commuters are less likely to be using the bikes. 

### Inventory:

- As mentioned above, there are some indicators that Ford GoBike is at least relatively efficient in their use of inventory. However, some improvement might be made. For instance, some of the most popular bike stations (measured in zero bike availability) have fewer than the maximum number of docks. This could be frustrating for riders. Ford GoBike should look into adding additional docking stations at these more popular (and possibly empty of bikes) stations. It is not possible to tell from the data set if they can add these, but it is worth looking into. 

- Assuming Ford GoBike implements at least some of these suggestions, they should be mindful that rider patterns (and therefore bike demands at any given location) are subject to change. Ford GoBike should be vigilant in keeping an eye on this and adjusting inventory usage (such as docking stations) accordingly.  

### Data Issues:

- Since the longest ride was for an entire month, Ford GoBike should look into ways in which they can prevent people from gaming the system. For instance, the vast majority of rides are under an hour duration, and very few are more than 4 hours. Capping the amount of time a rider can keep a bike at around the 4 hour mark could help with this. Otherwise, those bikes aren't making money for Ford GoBike when they are idle. 

- There are far too many (178692) rides shorter than 5 minutes. This is just over 18 percent  of all rides, which is highly problematic. The best assumption available from this data set is that this is due to user error. Ford GoBike should look into the reasons why this is so prevalent. If there is something unintuitive about the system, then riders are likely to be (understandably) frustrated. This frustration is likely to lead to fewer repeat customers and few positive recommendations. Fortunately, looking into this is something within Ford GoBike's control, and should be a fixable issue. 
