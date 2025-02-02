# Project Title :
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
The dataset were incosistent and SSIS was used for ETL (Extract, Transform and Load)

![Screenshot (141)](https://github.com/user-attachments/assets/5a9dc09e-4d82-4b1e-ae4a-07fe0caf6563)

![Screenshot (140)](https://github.com/user-attachments/assets/6e16550a-926e-4e1e-9d70-51716de1b53d)

![Screenshot (142)](https://github.com/user-attachments/assets/6fdbd6ae-c4df-4969-b1ad-f2d787f784ee)

---
### Data Cleaning

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

#### Insights
Users tend to listen to a wider variety of tracks when shuffle is OFF(Diasabled)

- What percentage of tracks played in shuffle mode are interrupted (reason_end)?

```
SELECT 
    COUNT(*) AS total_shuffle_plays,
    SUM(CASE WHEN Reaason_end <> 'trackdone' THEN 1 ELSE 0 END) AS interrupted_tracks,
 cast(round(SUM(CASE WHEN Reaason_end <> 'trackdone' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),2) as decimal(5,2)) AS interruption_rate
FROM New_clean_sportify_stream
WHERE shuffle = 'true';

```
![Screenshot (144)](https://github.com/user-attachments/assets/ae62a5c2-afa7-45e8-b485-fde98f583d1b)

- Which platforms have the highest shuffle mode usage?
```
SELECT platform, count(*) as total_play
FROM New_clean_sportify_stream
WHERE Shuffle = 'True'
GROUP BY platform
ORDER BY total_play desc

```
![Screenshot (145)](https://github.com/user-attachments/assets/8d3f7f8e-6a9c-4518-a176-8239b5680f43)

#### Insights
The highest shuffle mode was Andriod with 100,559 shuffle plays 

---

Track completion rates:
- What percentage of tracks are stopped early versus completed?
  
  ```
  WITH TrackCompletionStats AS (
    SELECT 
        COUNT(*) AS total_tracks_played,
        SUM(CASE WHEN reason_end = 'track_completed' THEN 1 ELSE 0 END) AS completed_tracks,
        SUM(CASE WHEN reason_end <> 'track_completed' THEN 1 ELSE 0 END) AS stopped_early
    FROM New_clean_sportify_stream)
   SELECT 
    total_tracks_played,
    completed_tracks,
    stopped_early,
    (stopped_early * 100.0 / total_tracks_played) AS stopped_early_percentage
    
   FROM TrackCompletionStats;

```
![Screenshot (146)](https://github.com/user-attachments/assets/16b9009e-d28f-47c3-9966-048ff9da90ab)


- Are there specific tracks or artists with consistently high interruption rates?






