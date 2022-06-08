Select *
FROM stream 
LIMIT 20;

SELECT *
FROM chat
LIMIT 20;

SELECT DISTINCT game
FROM stream;

SELECT DISTINCT channel
FROM stream;

SELECT game, COUNT(*)
FROM stream
GROUP BY game
ORDER BY COUNT(*) DESC;

SELECT country, COUNT(*)
FROM stream
WHERE game='League of Legends'
GROUP BY country
ORDER BY COUNT(*) DESC;

SELECT player, COUNT(*)
FROM stream
GROUP BY 1
ORDER BY 2 DESC;

SELECT game
,CASE 
WHEN game = 'League of Legends' THEN 'MOBA'
WHEN game = 'Dota 2' THEN 'MOBA'
WHEN game = 'Heroes of the Storm' THEN 'MOBA'
WHEN game = 'Counter-Strike: Global Offensive' THEN 'FPS'
WHEN game = 'DayZ' then 'Survival'
WHEN game = 'ARK: Survival Evolved' THEN 'Survival'
ELSE 'Other'
END AS 'Genre',
Count(*)
FROM stream
GROUP BY 1
ORDER BY 3 DESC

SELECT time,
strftime('%S', time)
FROM stream
GROUP BY 1
LIMIT 20;

SELECT strftime('%H', time), COUNT(*)
FROM stream
WHERE country='US'
GROUP BY 1;

With Twitch AS (
  SELECT *
FROM stream s
JOIN chat c 
  ON s.device_id=c.device_id
)
SELECT channel, subscriber,
Count(*) OVER (PARTITION BY channel ORDER BY subscriber desc) as '# of' 
FROM Twitch
