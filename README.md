# Olympics-SQL-Project
# SQL Codes:

### Tables:
SELECT * FROM noc;
SELECT * FROM Olympics;

### How many olympics games have been held?
SELECT COUNT(DISTINCT games)as Number_of_Games FROM Olympics;

### List down all Olympics games held so far.
SELECT games, city FROM Olympics
GROUP BY games, city
ORDER BY 1;

### Mention the total no of nations who participated in each olympics game?
SELECT o.games AS Games, COUNT(DISTINCT n.region) AS Number_of_Participating_Nations FROM olympics o 
JOIN noc n ON n.noc = o.noc 
GROUP BY 1;

### Which year saw the highest and lowest no. of countries participating in olympics.
SELECT highest.Highest_Participanting_Year, highest.Number_of_Countries,
lowest.Lowest_Participanting_Year, lowest.Number_of_Countries
FROM
(SELECT o.Year AS Highest_Participanting_Year, COUNT(DISTINCT n.region)AS Number_of_Countries FROM olympics o
JOIN noc n on n.noc = o.noc 
GROUP BY 1
ORDER BY 2 desc
LIMIT 1
) AS Highest
JOIN 
(SELECT o.Year AS Lowest_Participanting_Year, COUNT(DISTINCT n.region)AS Number_of_Countries FROM olympics o
JOIN noc n on n.noc = o.noc 
GROUP BY 1
ORDER BY 2 
LIMIT 1
) AS Lowest
ON 1=1

### Which nation has participated in all of the olympic games.
WITH total_games as
(SELECT COUNT(DISTINCT games)AS total_no_of_games FROM Olympics),
    total_countries AS
(SELECT o.games,n.region AS Country FROM Olympics o
JOIN noc n ON n.noc = o.noc 
GROUP BY 1,2
ORDER BY 1),
    country_participated AS
(SELECT country, COUNT(1)AS total_Participants FROM total_countries
GROUP BY 1)
SELECT cp.* FROM country_participated cp
JOIN total_games tg ON tg.total_no_of_games = cp.total_Participants

### Identify the sport which was played in all summer olympics.
WITH t1 AS
(SELECT COUNT(DISTINCT games)AS Total_Count_of_Games FROM Olympics
WHERE Season = 'Summer'),
    t2 AS 
(SELECT DISTINCT sport, games FROM Olympics 
 WHERE Season = 'Summer'
 GROUP BY 2,1
 ORDER BY 2),
     t3 AS 
(SELECT sport, COUNT(Games)as No_of_Games FROM t2
GROUP BY 1) 
SELECT sport, No_of_Games FROM t3
JOIN t1 ON t1.Total_Count_of_Games = t3.No_of_Games
 
### Which Sports were just played only once in the olympics?
SELECT Sport, COUNT(DISTINCT Games) FROM Olympics
GROUP BY 1
HAVING COUNT(DISTINCT Games) = 1;

### Fetch the total no of sports played in each olympic games.
SELECT games, COUNT(DISTINCT Sport)AS Number_of_Games_Played FROM Olympics
GROUP BY games
ORDER BY 2 DESC;

### Fetch details of the oldest athletes to win a gold medal.
WITH Old AS
(SELECT *,
Rank() OVER(ORDER BY age DESC)AS Rank
FROM Olympics
WHERE medal = 'Gold' AND age NOT LIKE 'NA')
SELECT * FROM Old 
WHERE Rank = 1;

### Find the Ratio of male and female athletes participated in all olympic games.
WITH t1 AS (
    SELECT sex, COUNT(1) AS cnt
    FROM olympics
    GROUP BY sex
),
min_max_counts AS (
    SELECT MIN(cnt) AS min_count, MAX(cnt) AS max_count
    FROM t1
)

SELECT CONCAT('1 : ', ROUND(max_count::decimal / min_count, 2)) AS ratio
FROM min_max_counts;

### Fetch the top 5 athletes who have won the most gold medals.
SELECT name, team, COUNT(medal) AS num_of_gold_medal 
FROM Olympics
WHERE medal = 'Gold'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5;

### Fetch the top 5 athletes who have won the most medals (gold/silver/bronze).
SELECT name, team, COUNT(medal)AS num_of_medals 
FROM Olympics
WHERE medal in ('Gold','Silver','Bronze')
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5;

### Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.
SELECT n.region AS Country, COUNT(o.medal)AS num_of_medals 
FROM Olympics o
JOIN noc n ON n.noc = o.noc
WHERE o.medal in ('Gold','Silver','Bronze')
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

### List down total gold, silver and broze medals won by each country.
SELECT n.region as COUNTRY, 
 SUM(CASE WHEN o.medal = 'Gold' THEN 1 ELSE 0 END)AS Gold,
 SUM(CASE WHEN o.medal = 'Silver' THEN 1 ELSE 0 END)AS Silver,
 SUM(CASE WHEN o.medal = 'Bronze' THEN 1 ELSE 0 END)AS Bronze
FROM Olympics o 
JOIN noc n ON n.noc = o.noc 
GROUP BY 1
ORDER BY 2 DESC,3 DESC,4 DESC;

### List down total gold, silver and broze medals won by each country corresponding to each olympic games.
SELECT o.games, n.region AS Country, 
SUM(CASE WHEN o.medal LIKE 'Gold' THEN 1 ELSE 0 END) AS Gold,
SUM(CASE WHEN o.medal LIKE 'Silver' THEN 1 ELSE 0 END)AS Silver,
SUM(CASE WHEN o.medal LIKE 'Bronze' THEN 1 ELSE 0 END)AS Bronze
FROM Olympics o 
JOIN noc n ON n.noc = o.noc 
GROUP BY 1,2
ORDER BY 1,2,3 DESC,4 DESC,5 DESC;

### Which countries have never won gold medal but have won silver/bronze medals?
SELECT n.region AS Country,
    SUM(CASE WHEN o.medal = 'Gold' THEN 1 ELSE 0 END) AS Total_Gold_Medals,
    SUM(CASE WHEN o.medal = 'Silver' THEN 1 ELSE 0 END) AS Total_Silver_Medals,
    SUM(CASE WHEN o.medal = 'Bronze' THEN 1 ELSE 0 END) AS Total_Bronze_Medals
FROM Olympics o 
JOIN noc n ON n.noc = o.noc 
GROUP BY n.region
HAVING SUM(CASE WHEN o.medal = 'Gold' THEN 1 ELSE 0 END) = 0
   AND (SUM(CASE WHEN o.medal = 'Silver' THEN 1 ELSE 0 END) > 0 OR SUM(CASE WHEN o.medal = 'Bronze' THEN 1 ELSE 0 END) > 0)
ORDER BY 2 DESC, 3 DESC;

### In which Sport/event, India has won highest medals.
SELECT o.sport,n.region, COUNT(o.medal) AS num_of_medals
FROM Olympics o
JOIN noc n ON n.noc = o.noc 
WHERE n.region = 'India' AND medal NOT LIKE 'NA'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1;

###Break down all olympic games where india won medal for Hockey and how many medals in each olympic games.
SELECT games, team, sport, COUNT(medal)AS num_of_medals 
FROM Olympics
WHERE Team = 'India' AND Sport = 'Hockey' AND medal <> 'NA'
GROUP BY 1,2,3
ORDER BY 4 DESC;
