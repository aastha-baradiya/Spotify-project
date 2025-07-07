# Spotify-project
CREATE TABLE SPOTIFY_DATABASE (
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
/*
Easy Level
Retrieve the names of all tracks that have more than 1 billion streams.
List all albums along with their respective artists.
Get the total number of comments for tracks where licensed = TRUE.
Find all tracks that belong to the album type single.
Count the total number of tracks by each artist.
*/

--Q.1 Retrieve the names of all tracks that have more than 1 billion streams.

SELECT * FROM SPOTIFY_DATABASE WHERE stream > 1000000000;

--Q.2 List all albums along with their respective artists.

SELECT DISTINCT album, artist FROM SPOTIFY_DATABASE ORDER BY 1;

--Q.3 Get the total number of comments for tracks where licensed = TRUE.

SELECT SUM(comments) FROM SPOTIFY_DATABASE WHERE licensed = 'TRUE';

--Q.4 Find all tracks that belong to the album type single.

SELECT * FROM SPOTIFY_DATABASE WHERE album_type = 'single';

--Q.5 Count the total number of tracks by each artist.

SELECT artist, COUNT(track) FROM SPOTIFY_DATABASE GROUP BY 1 ORDER BY 2 DESC;

/*
Medium Level
Calculate the average danceability of tracks in each album.
Find the top 5 tracks with the highest energy values.
List all tracks along with their views and likes where official_video = TRUE.
For each album, calculate the total views of all associated tracks.
Retrieve the track names that have been streamed on Spotify more than YouTube.
*/

--Q.6 Calculate the average danceability of tracks in each album.

SELECT album, AVG(danceability) FROM SPOTIFY_DATABASE GROUP BY 1;

--Q.7 Find the top 5 tracks with the highest energy values.

SELECT track, MAX(energy) FROM SPOTIFY_DATABASE GROUP BY 1 ORDER BY 2 DESC LIMIT 5;

--Q.8 List all tracks along with their views and likes where official_video = TRUE.

SELECT track, SUM(VIEWS) AS TOTAL_VIEWS, SUM(LIKES) AS TOTAL_LIKES
FROM SPOTIFY_DATABASE WHERE official_video = 'TRUE' GROUP BY 1;

--Q.9 For each album, calculate the total views of all associated tracks.

SELECT album, TRACK, SUM (VIEWS) AS TOTAL_VIEWS FROM SPOTIFY_DATABASE GROUP BY 1,2 ORDER BY 3 DESC;

--Q.10 Retrieve the track names that have been streamed on Spotify more than YouTube.

SELECT * FROM
(SELECT track,
COALESCE (SUM(CASE WHEN most_played_on = 'Youtube' THEN STREAM END),0) AS YOUTUBE,
COALESCE (SUM(CASE WHEN most_played_on = 'Spotify' THEN STREAM END),0) AS SPOTIFY
FROM SPOTIFY_DATABASE GROUP BY 1)
WHERE SPOTIFY>YOUTUBE AND YOUTUBE<>0;

/*
Advanced Level
Find the top 3 most-viewed tracks for each artist using window functions.
Write a query to find tracks where the liveness score is above the average.
Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album
Find tracks where the energy-to-liveness ratio is greater than 1.2.
Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.
*/

--Q.11 Find the top 3 most-viewed tracks for each artist using window functions.

WITH ARTIST_RANK AS
(SELECT artist, track, SUM(views), 
DENSE_RANK () OVER(PARTITION BY artist ORDER BY SUM(views) DESC) AS RANK
FROM SPOTIFY_DATABASE GROUP BY 1,2 ORDER BY 1,3 DESC)
SELECT * FROM ARTIST_RANK
WHERE RANK<=3;

--Q.12 Write a query to find tracks where the liveness score is above the average.

SELECT track, liveness FROM SPOTIFY_DATABASE
WHERE liveness > (SELECT AVG(liveness) FROM SPOTIFY_DATABASE) ORDER BY 2 ASC;

--Q.13 Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album

WITH ABC AS 
(SELECT album,
MAX(energy) AS HIGHEST_ENERGY,
MIN(energy) AS LOWEST_ENERGY
FROM SPOTIFY_DATABASE GROUP BY 1)
SELECT album, HIGHEST_ENERGY - LOWEST_ENERGY AS ENERGY_DIFF FROM ABC ORDER BY 2 DESC;

--Q.14 Find tracks where the energy-to-liveness ratio is greater than 1.2.

SELECT track, energy_liveness FROM SPOTIFY_DATABASE WHERE energy_liveness>1.2 ORDER BY 2 DESC;

--Q.15 Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.

SELECT track, views, likes, SUM(likes)
OVER (ORDER BY views DESC) AS cumulative_likes
FROM SPOTIFY_DATABASE
ORDER BY views DESC;
