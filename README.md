# Spotify SQL Project and Query Optimization 


## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity  and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
---SQL PROJECT SPOTIFY--

-- Create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
--EDA
```sql
SELECT COUNT (*) FROM spotify;

SELECT COUNT (DISTINCT artist) FROM spotify;

SELECT DISTINCT album_type FROM spotify;

SELECT MAX (duration_min) FROM spotify;

SELECT MIN(duration_min) FROM spotify;

SELECT* FROM spotify
WHERE duration_min = 0;
SELECT*FROM spotify
WHERE duration_min = 0;

SELECT DISTINCT channel FROM spotify;
SELECT DISTINCT most_played_on FROM spotify;
```

-----------


--

--Q.1.Retrieve the names of all tracks that have more than 1 billion streams.

```sql
SELECT*FROM spotify
WHERE stream > 1000000000
```

--Q.2.List all albums along with their respective artists.
```sql
SELECT
	DISTINCT album,artist
FROM spotify
ORDER BY 1

SELECT 	
	DISTINCT album
FROM spotify
ORDER BY 1
```
--Q.3 Get the total number of comments for tracks where licensed = TRUE.

```sql

SELECT
	SUM(comments) as total_comments 
FROM spotify
WHERE licensed ='true'
```
--Q.4 Find all tracks that belong to the album type single.
```sql

SELECT*FROM spotify
WHERE album_type ='single'
```
--Q.5 Count the total number of tracks by each artist.
```sql
SELECT
	artist,---1
	COUNT(*) as total_no_songs --2
FROM spotify
GROUP BY artist
ORDER BY 2
```
--Q.6 Calculate the average danceability of tracks in each album.
```sql
SELECT
	album,
	avg(danceability) as avg_daceability
FROM spotify
GROUP BY 1
ORDER BY 2 DESC
```
--Q.7 Find the top 5 tracks with the highest energy values.
```sql
SELECT
	track,
	AVG(energy)
FROM spotify
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```
--Q.8 List all tracks along with their views and likes where official_video = TRUE.
```sql

SELECT
	track,
	SUM(views) as total_views,
	SUM(likes) as total_views
FROM spotify
WHERE official_video ='true'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

--Q.9 For each album, calculate the total views of all associated tracks.
```sql

SELECT 
	album,
	track,
	SUM(views)
FROM spotify
GROUP BY 1,2
ORDER BY 3 DESC
```
--Q.10 Retrieve the track names that have been streamed on Spotify more than YouTube.
```sql

SELECT*FROM
(SELECT
	track,
	--most_played_on,
	COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' THEN stream END),0) as streamed_on_youtube,
	COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END),0) as streamed_on_spotify
FROM spotify
GROUP BY 1
) as t1
WHERE 
	streamed_on_spotify > streamed_on_youtube
	AND
	streamed_on_youtube <> 0
```
--Q.11 Find the top 3 most-viewed tracks for each artist using window functions.
```sql

WITH ranking_artist
AS
(SELECT 
	artist,
	track,
	SUM(views) as total_view,
	DENSE_RANK()OVER(PARTITION BY artist ORDER BY SUM(views)DESC) as rank
FROM spotify
GROUP BY 1 ,2
ORDER BY 1,3 DESC
)
SELECT *FROM ranking_artist
WHERE rank <=3
```
--12.Write a query to find tracks where the liveness score is above the average.
```sql
SELECT
	track,
	artist,
	liveness
FROM spotify
WHERE liveness > (SELECT AVG (liveness) FROM spotify)
```
--Q.13 Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.
```sql
WITH cte
AS 
(SELECT 
	album,
	MAX(energy)as highest_energy,
	MIN(energy)as lowest_energy
FROM spotify
GROUP BY 1 
)
SELECT
	album,
	highest_energy - lowest_energy as energy_diff
FROM cte
ORDER BY 2 DESC
```

--Query Optimization

```sql
EXPLAIN ANALYZE 
SELECT
	artist,
	track,
	views
FROM spotify
WHERE artist = 'Gorillaz'
	AND
	most_played_on = 'Youtube'
	ORDER BY stream DESC LIMIT 25

```
## Query Optimization Technique 

To improve query performance, I carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - I began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
   



This optimization shows how indexing can drastically reduce query time, improving the overall performan
---

## License
This project is licensed under the MIT License.
