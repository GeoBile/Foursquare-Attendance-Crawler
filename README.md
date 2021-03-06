FoursquareAttendanceCrawler
=============================

If you use this code for a research purpose, please use the following citation:

Romain Deveaud, M-Dyaa Albakour, Craig Macdonald, and Iadh Ounis. *Experiments with a Venue-Centric Model for Personalised and Time-Aware Venue Suggestion.* In CIKM 2015, Shanghai, China. http://dl.acm.org/citation.cfm?id=2806484

Bibtex:
```
@inproceedings{Deveaud:2015:EVM:2806416.2806484,
 author = {Deveaud, Romain and Albakour, M-Dyaa and Macdonald, Craig and Ounis, Iadh},
 title = {Experiments with a Venue-Centric Model for Personalisedand Time-Aware Venue Suggestion},
 booktitle = {Proceedings of the 24th ACM International on Conference on Information and Knowledge Management},
 series = {CIKM '15},
 year = {2015},
 isbn = {978-1-4503-3794-6},
 location = {Melbourne, Australia},
 pages = {53--62},
 numpages = {10},
 url = {http://doi.acm.org/10.1145/2806416.2806484},
 doi = {10.1145/2806416.2806484},
 acmid = {2806484},
 publisher = {ACM},
 address = {New York, NY, USA},
 keywords = {location-based social networks, personalisation, time series forecasting, venue recommendation},
} 
```


## Settings

Open the `etc/settings-example.json` file and change the `foursquare_api_accounts` and `crawl_folder` properties. You can obtain credentials for the Foursquare API by creating an application at https://foursquare.com/developers/apps.

After setting the parameters to the appropriate values, change the name of the file to `etc/settings.json`, otherwise the program will not find it:

```
  $ mv etc/settings-example.json etc/settings.json
```


## Selecting the venues to crawl

The first step is to scan the entire city in order to obtain a pool of all the venues of the city. Only venues with more than 25 overall checkins will be kept at this stage. To do this, you need to execute the `GetAllVenues` program:

```
  $ java -Dfile.encoding=UTF-8 -classpath bin:lib/commons-io-2.4.jar:lib/commons-lang-2.6.jar:lib/gson-1.7.1.jar eu.smartfp7.foursquare.GetAllVenues london
```

This step will likely take a lot of time (~12-24 hours, depending on the size of the city), and will not end until you cancel the execution of the program. Do not cancel the execution if you haven't seen the following message:

> Finished crawling everything. Pausing 10 minutes before restarting...

The resulting list of venues will be written in a `.exhaustive_crawl` folder, created under the directory that you specified in the `etc/settings.json` file. Since the attendance crawler cannot deal with more than 5,000 venues, we need to filter the set of all venues:

```
  $ java -Dfile.encoding=UTF-8 -classpath bin:lib/commons-io-2.4.jar:lib/commons-lang-2.6.jar:lib/gson-1.7.1.jar eu.smartfp7.foursquare.FilterVenues london
```

This program will keep the 3,000 venues with the most number of checkins (i.e. the most popular venues), and will select 1,950 more venues from the remaining ones. This set of 4,950 venues will be the venues for which we will obtain hourly levels of attendance.



## Crawling the hourly attendance of venues

Simply run this program to start the crawling for all the venues:

```
  $ java -Dfile.encoding=UTF-8 -classpath bin:lib/commons-io-2.4.jar:lib/commons-lang-2.6.jar:lib/gson-1.7.1.jar eu.smartfp7.foursquare.AttendanceCrawler london
```

One file per venue will be created in the `attendances_crawl` directory, where each line corresponds to one observation per hour. These files can be read and parsed using the `RTimeSeries` class.

If you have several cities in your `settings.json` file, you can also launch the crawling for all of them at once:

```
  $ java -Dfile.encoding=UTF-8 -classpath bin:lib/commons-io-2.4.jar:lib/commons-lang-2.6.jar:lib/gson-1.7.1.jar eu.smartfp7.foursquare.AttendanceCrawler london amsterdam sanfrancisco shanghai glasgow
```



## What if the crawler misses some hours?

Sometimes the Foursquare API service can experience some issues, and the crawler can miss the hourly attendance of a venue, which creates sparse time series (and this is not good).
This is why the crawler only considers 4,950 venues while it has 5,000 calls per hour: the remaining 50 calls can be used to attempt to get the hourly attendance of a venue a second time, if it failed in the first instance.

However, sometimes this is not enough, and the crawler may miss a large number of hourly observations.
The crawler includes an automatic process for reconstructing these missing points, which runs every day at ~12:30am.
This process can reconstruct two types of missing points:
  1. when only a single point is missing, it performs a simple linear interpolation (http://en.wikipedia.org/wiki/Linear_interpolation). This can result with decimal numbers, but this is fine for the purpose of showing/using time series data.
  2. when a sequence of points are missing, it uses a very strong baseline in time series forecasting, which simply takes the value of the time series at the same hour the day before. This is called seasonal naive forecasting (https://www.otexts.org/fpp/2/3).



## How to predict the future attendance of a venue?

Using the data generated by the attendance crawler, we can use statistical methods to predict the attendance of a venue at future points in time.

See https://github.com/SmartSearch/Foursquare-Attendance-Forecasting !
