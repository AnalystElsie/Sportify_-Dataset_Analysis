## Project Title:
Optimizing Shuffle Mode & Track Completion Rates for Enhanced User Engagement on Spotify

---


### Project Overview
The project aims to analyze user behavior on Spotify, specifically focusing on shuffle mode and track completion rates, to provide insights that will help Spotify improve user engagement and optimize the shuffle feature. By understanding the impact of shuffle mode, track interruptions, and platform usage trends, the goal is to identify areas for improvement that will encourage users to engage more with the platform, enhance track completion rates, and optimize the overall streaming experience.

---

### Data Sources
The dataset used for this analysis is the "Sportify_history.csv" 

---
### Tools
- SSIS - for data Migration into the database
- SSMS- to query the database
- PowerBi- Creating Reports

---
### Data Dictionary
The dictionary file is "Sportify_data_dictionary.csv"

---

### Migration of data
The dataset were incosistent and SSIS was used for ETL (Extract, Transform and Load) and a total of 149,980 rows were iimported

![Screenshot (141)](https://github.com/user-attachments/assets/5a9dc09e-4d82-4b1e-ae4a-07fe0caf6563)

![Screenshot (140)](https://github.com/user-attachments/assets/6e16550a-926e-4e1e-9d70-51716de1b53d)

![Screenshot (142)](https://github.com/user-attachments/assets/6fdbd6ae-c4df-4969-b1ad-f2d787f784ee)

---
### Data Cleaning
A copy of the data was made New_clean_sportify_streams
```
 select * into New_clean_sportify_stream
 From
 (
 select spotify_track_url, ts,platform, round(ms_played/60000.0, 2) as ms_played_minuites, track_name,artist_name,album_name,
 case
 when reason_start = ' ' then null else reason_start end as Reason_start,
 case
 when reason_end = ' ' then null else reason_end end as Reaason_end,
 case
 when shuffle = 1 then 'True' else 'False' end as Shuffle,
 case 
 when skipped =1 then 'True' else 'False' end as Skipped
 from Clean_sportify_stream
)x

```

------

### Business Questions
Impact of shuffle mode on listening behaviour:
- Do users play a more diverse range of tracks when shuffle mode is enabled?
  
```
SELECT 
shuffle,
COUNT(DISTINCT track_name) AS unique_tracks_played
FROM New_clean_sportify_stream
GROUP BY shuffle;

```
![Screenshot (143)](https://github.com/user-attachments/assets/1bd12bda-b395-46e3-bfbb-bddaea29ac27)

### Insights
Users tend to listen to a wider variety of tracks when shuffle is OFF(Diasabled)

- What percentage of tracks played in shuffle mode are interrupted (reason_end)?

```
SELECT
shuffle,
COUNT(*) AS total_shuffle_plays,
SUM(CASE WHEN Reaason_end <> 'trackdone' THEN 1 ELSE 0 END) AS interrupted_tracks,
cast(round(SUM(CASE WHEN Reaason_end <> 'trackdone' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),2) as decimal(5,2)) AS interruption_rate
FROM New_clean_sportify_stream
WHERE shuffle = 'true';

```
![Screenshot (148)](https://github.com/user-attachments/assets/274bc282-7cf5-412c-be9f-4fb68b562daa)

### insights
The percentage of interuption rate is higher when shuffle mode is enabled(54.04%) as compared to when its turned off (39.90%)


- Which platforms have the highest shuffle mode usage?
```
SELECT platform, shuffle, count(*) as total_play
FROM New_clean_sportify_stream
WHERE Shuffle = 'True'
GROUP BY platform
ORDER BY total_play desc

```
![Screenshot (149)](https://github.com/user-attachments/assets/d0ab1201-8f09-4c62-9e9b-b0c86a64d1c9)


#### Insights
The highest shuffle mode was Andriod with 100,559 shuffle plays when enabled and 39,262 when disabled 

---

Track completion rates:
- What percentage of tracks are stopped early versus completed?

```
WITH TrackCompletionStats AS (
SELECT 
COUNT(*) AS total_tracks_played,
SUM(CASE WHEN reason_end = 'track_completed' THEN 1 ELSE 0 END)
AS completed_tracks,
SUM(CASE WHEN reason_end <> 'track_completed' THEN 1 ELSE 0 END) AS stopped_early
FROM New_clean_sportify_stream)
SELECT 
total_tracks_played,
completed_tracks,
stopped_early,
cast( round(stopped_early * 100.0 / total_tracks_played,2)as decimal(5,2)) AS stopped_early_percentage ,
cast(round(completed_tracks *100.0/ total_tracks_played, 2)as decimal(5,2)) As completed_percentage
FROM TrackCompletionStats;

```
![Screenshot (147)](https://github.com/user-attachments/assets/b31eef03-dac8-4d90-8b45-b202aa49e58d)


### Insights
The percentage of tracks stopped early was 49.72% compared to completed which is 50.20%.

- Are there specific tracks or artists with consistently high interruption rates?

```
 SELECT 
track_name, 
artist_name,
COUNT(*) AS total_plays,
SUM(CASE WHEN Reaason_end
<>'trackdone' THEN 1 ELSE 0 END) AS interrupted_count,
(SUM(CASE WHEN Reaason_end <> 'trackdone' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)) AS interruption_rate
FROM New_clean_sportify_stream
GROUP BY track_name, artist_name
HAVING COUNT(*) > 10  
ORDER BY interruption_rate DESC, artist_name

```
### Insights

- Does the platform or shuffle mode influence track completion rates?
```
select platform, shuffle,
count(*) as total_played,
sum(case when Reaason_end = 'trackdone' then 1 else 0 end) as completion_cont,
cast(sum(case when Reaason_end = 'trackdone' then 1 else 0 end) *100.0 /count(*) as decimal(5,2)) as completion_rate
from New_clean_sportify_stream
group by platform, Shuffle
order by completion_rate desc

```
![Screenshot (150)](https://github.com/user-attachments/assets/2f7d444b-2f49-4187-a77f-ab3d7dbcb63c)

### Insights
Platform completion rate is higher when shufle mode is enabled

---

Platform usage trends:
- Which platforms have the longest average playback duration?
```
select platform, round(avg(ms_played_minuites),2)as  average_play_back
from New_clean_sportify_stream
group by platform

```
![Screenshot (151)](https://github.com/user-attachments/assets/751d91ce-8c98-49ed-a3b0-5c4c638aec9d)

### insights
Mac platform has the highest avaerage playback with 3:57mins

--Are there specific hours or days where platform usage peaks?

```
select DATEPART(hour,ts) as hours, count(*)total_usage
from New_clean_sportify_stream
group by DATEPART(hour,ts)
order by total_usage desc

```
![Screenshot (153)](https://github.com/user-attachments/assets/cb0980ee-434b-401b-9a82-64146d6b5fe3)

### Insights
At 0hrs the peak usage 10884

Timestamp based insights:
- What are the most popular hours for streaming across different platforms?

```
select platform,DATEPART(hour,ts) as hours, count(*)total_usage
from New_clean_sportify_stream
group by platform,DATEPART(hour,ts)
order by total_usage desc

```
![Screenshot (152)](https://github.com/user-attachments/assets/c34467dd-57b6-44e6-a7a6-de4a5b783ae1)

### Insights
- At midnight (0hrs) andriod recorded the highest usage at 10,466 streams.
- At 5pm( 17hrs) Cast device recorded highest usage at 677 streams.
- AT 6pm (18hrs) IOS recorded recorded hightes usage at 317 streams.
  
  ---

- Which tracks are most frequently played during peak hours?
  
```
WITH PeakHours AS (
SELECT top 1  DATEPART(HOUR, ts) AS peak_hour
FROM New_clean_sportify_stream
GROUP BY DATEPART(HOUR, ts)
ORDER BY COUNT(*) DESC

)
SELECT top 10
track_name, 
COUNT(*) AS play_count
FROM New_clean_sportify_stream
WHERE DATEPART(HOUR, ts) = (SELECT peak_hour FROM PeakHours)
GROUP BY track_name
ORDER BY play_count DESC

```

![Screenshot (154)](https://github.com/user-attachments/assets/287893c9-3cac-4c33-be3e-19c2d40fae36)

