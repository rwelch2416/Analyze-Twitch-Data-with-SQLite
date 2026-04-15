# Analyze-Twitch-Data-with-SQLite
Analyzing Twitch data from two different time periods using SQLite. The data is based on users language, activity, popularity and etc.
⚙️ Setup
1. Clone this repo
```bash
git clone https://github.com/your-username/twitch-sqlite-analysis.git
cd twitch-sqlite-analysis
```
2. Open SQLite and create your database
```bash
sqlite3 twitch.db
```
3. Import both CSV files
```sql
.mode csv
.import streamers2021.csv streamers2021
.import streamers2024.csv streamers2024
```
4. Verify the import
```sql
.tables
SELECT * FROM streamers2021 LIMIT 3;
SELECT * FROM streamers2024 LIMIT 3;
```
> **Windows tip:** If SQLite can't find your CSV, either `cd` into the folder containing the file before launching SQLite, or use the full path with forward slashes:
> `.import C:/Users/YourName/Downloads/streamers2021.csv streamers2021`
---
🔍 SQL Queries
```sql
-- Top 10 streamers by watch time in 2021
SELECT channel, watch_time, average_viewers, language
FROM streamers2021
ORDER BY watch_time DESC
LIMIT 10;

-- Top 10 streamers by total followers in 2024
SELECT name, total_followers, language, most_streamed_game
FROM streamers2024
ORDER BY total_followers DESC
LIMIT 10;
```
---
JOIN — Streamers in Both Years
Here it finds streamers who appear in both the 2021 and 2024 top lists.
```sql
-- How many top streamers from 2021 are still in the top in 2024?
SELECT COUNT(*) AS streamers_in_both_years
FROM streamers2021 a
JOIN streamers2024 b
  ON LOWER(a.channel) = LOWER(b.name);
-- Result: 265
```
```sql
-- Full list with follower growth
SELECT a.channel AS streamer,
       a.language,
       a.followers        AS followers_2021,
       b.total_followers  AS followers_2024,
       b.total_followers - a.followers AS follower_growth
FROM streamers2021 a
JOIN streamers2024 b
  ON LOWER(a.channel) = LOWER(b.name)
ORDER BY follower_growth DESC;
```
---
LEFT JOIN — Streamers Who Dropped Off
This finds streamers who were in the 2021 top list but did not make the 2024 list.
```sql
-- Streamers in 2021 who are NOT in 2024 (dropped off)
SELECT a.channel, a.followers, a.language
FROM streamers2021 a
LEFT JOIN streamers2024 b
  ON LOWER(a.channel) = LOWER(b.name)
WHERE b.name IS NULL
ORDER BY a.followers DESC;
-- Result: 735 streamers
```
```sql
-- New entrants in 2024 who were NOT in 2021
SELECT b.name, b.total_followers, b.language
FROM streamers2024 b
LEFT JOIN streamers2021 a
  ON LOWER(a.channel) = LOWER(b.name)
WHERE a.channel IS NULL
ORDER BY b.total_followers DESC;
-- Result: 734 streamers
```
---
```sql
-- All streamers from both years in one list (no duplicates by name)
SELECT LOWER(channel) AS streamer, language, 'English' AS source_year
FROM streamers2021
WHERE language = 'English'

UNION

SELECT LOWER(name) AS streamer, language, '2024' AS source_year
FROM streamers2024
WHERE language = 'English'

ORDER BY streamer;
```
```sql
-- Language popularity: combined count across both years
SELECT language, COUNT(*) AS total_streamers, '2021' AS year
FROM streamers2021
GROUP BY language

UNION ALL

SELECT language, COUNT(*) AS total_streamers, '2024' AS year
FROM streamers2024
GROUP BY language

ORDER BY year, total_streamers DESC;
---
Language Breakdown (Streamers in Both Years)
```sql
SELECT a.language,
       COUNT(*) AS streamers_in_both,
       ROUND(AVG(b.total_followers - a.followers), 0) AS avg_follower_growth
FROM streamers2021 a
JOIN streamers2024 b
  ON LOWER(a.channel) = LOWER(b.name)
GROUP BY a.language
ORDER BY avg_follower_growth DESC;
```
---
# 📊 Key Findings 
Streamers in both 2021 & 2024 top lists - 265 (26.5% rentention)

Streamers who dropped off - 735

New entrants in 2024 - 734

Highest follower growth - Ibai (+13.7M)

Language with highest avg growth - Spanish (+3.4M avg)

Most represented language - English (141 streamers in both years)
