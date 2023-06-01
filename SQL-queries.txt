--------------------------------------------------------------------------------------
--I asked 10 questions and answer them to understand better the following dataset.
--------------------------------------------------------------------------------------


--1. Which leagues, from which countries are included in the database?
SELECT
	l.name AS "League",
	c.name AS "Country"
FROM League l  
LEFT JOIN Country c  ON l.country_id = c.id;

-- Output:
League                  |Country    |
------------------------+-----------+
Belgium Jupiler League  |Belgium    |
England Premier League  |England    |
France Ligue 1          |France     |
Germany 1. Bundesliga   |Germany    |
Italy Serie A           |Italy      |
Netherlands Eredivisie  |Netherlands|
Poland Ekstraklasa      |Poland     |
Portugal Liga ZON Sagres|Portugal   |
Scotland Premier League |Scotland   |
Spain LIGA BBVA         |Spain      |
Switzerland Super League|Switzerland|


--2. How many goals are scored in each league and season (between 2008 and 2016)?
SELECT
	l.name AS "league",
	season,
	SUM(home_team_goal) + SUM(away_team_goal) AS "all goals"
FROM League l  
LEFT JOIN "Match" m  ON l.id = m.league_id
GROUP BY league, season
ORDER BY "all goals" DESC, league
LIMIT 10;

-- Output:
league                |season   |all goals|
----------------------+---------+---------+
Spain LIGA BBVA       |2008/2009|     1101|
Spain LIGA BBVA       |2012/2013|     1091|
England Premier League|2011/2012|     1066|
England Premier League|2010/2011|     1063|
England Premier League|2012/2013|     1063|
England Premier League|2009/2010|     1053|
England Premier League|2013/2014|     1052|
Spain LIGA BBVA       |2011/2012|     1050|
Spain LIGA BBVA       |2013/2014|     1045|
Spain LIGA BBVA       |2015/2016|     1043|


--3. What is the difference in the number of goals scored in home matches and away matches (between 2008 and 2016)?
SELECT
    l.name AS "league",
    season,
    SUM(home_team_goal) AS "home team goals",
    SUM(away_team_goal) AS "away team goals",
    ROUND(CAST(SUM(away_team_goal) AS FLOAT(1,2)) / CAST(SUM(home_team_goal) AS FLOAT(1,2)), 2) AS "home goals ratio to away goals"
FROM
    League l
    LEFT JOIN "Match" m ON l.id = m.league_id
GROUP BY
    l.name, season
ORDER BY
    "home team goals" DESC, "away team goals" DESC;

-- Output (first 10 rows):
league                  |season   |home team goals|away team goals|home goals ratio to away goals|
------------------------+---------+---------------+---------------+------------------------------+
England Premier League  |2009/2010|            645|            408|                          0.63|
Spain LIGA BBVA         |2012/2013|            641|            450|                           0.7|
Spain LIGA BBVA         |2011/2012|            638|            412|                          0.65|
Spain LIGA BBVA         |2008/2009|            631|            470|                          0.74|
Spain LIGA BBVA         |2010/2011|            622|            420|                          0.68|
Spain LIGA BBVA         |2013/2014|            620|            425|                          0.69|
England Premier League  |2010/2011|            617|            446|                          0.72|
Spain LIGA BBVA         |2015/2016|            615|            428|                           0.7|
Spain LIGA BBVA         |2009/2010|            608|            423|                           0.7|
England Premier League  |2011/2012|            604|            462|                          0.76|


--4. What is the number of wins, losses and draws for each team in each season (between 2008 and 2016)?
SELECT
    m.season,
    l.name AS league,
    t.team_long_name AS team,
    COUNT(*) AS total_matches,
    SUM(CASE WHEN m.home_team_goal > m.away_team_goal THEN 1 ELSE 0 END) AS wins,
    SUM(CASE WHEN m.home_team_goal < m.away_team_goal THEN 1 ELSE 0 END) AS losses,
    SUM(CASE WHEN m.home_team_goal = m.away_team_goal THEN 1 ELSE 0 END) AS draws
FROM
    match m
    JOIN league l ON l.id = m.league_id
    JOIN team t ON t.team_api_id = m.home_team_api_id
GROUP BY
    m.season,
    l.name,
    t.team_long_name
ORDER BY
    m.season,
    l.name,
    t.team_long_name;

-- Output (first 10 rows):
season   |league                  |team                        |total_matches|wins|losses|draws|
---------+------------------------+----------------------------+-------------+----+------+-----+
2008/2009|Belgium Jupiler League  |Beerschot AC                |           17|   9|     5|    3|
2008/2009|Belgium Jupiler League  |Club Brugge KV              |           17|  11|     4|    2|
2008/2009|Belgium Jupiler League  |FCV Dender EH               |           17|   5|     8|    4|
2008/2009|Belgium Jupiler League  |KAA Gent                    |           17|   9|     5|    3|
2008/2009|Belgium Jupiler League  |KRC Genk                    |           17|   7|     5|    5|
2008/2009|Belgium Jupiler League  |KSV Cercle Brugge           |           17|   9|     6|    2|
2008/2009|Belgium Jupiler League  |KSV Roeselare               |           17|   6|     7|    4|
2008/2009|Belgium Jupiler League  |KV Kortrijk                 |           17|   5|     6|    6|
2008/2009|Belgium Jupiler League  |KV Mechelen                 |           17|   6|     4|    7|
2008/2009|Belgium Jupiler League  |KVC Westerlo                |           17|  10|     5|    2|


--5. Which teams were the best in a given league and season (most wins)?
SELECT
    m.season,
    l.name AS league,
    MAX(t.team_long_name) AS best_team,
    MAX(wins) AS max_wins
FROM
    match m
    JOIN league l ON l.id = m.league_id
    JOIN team t ON t.team_api_id = m.home_team_api_id
    JOIN (
        SELECT
            season,
            league_id,
            home_team_api_id,
            COUNT(*) AS wins
        FROM
            match
        WHERE
            home_team_goal > away_team_goal
        GROUP BY
            season,
            league_id,
            home_team_api_id
    ) AS subquery ON subquery.season = m.season AND subquery.league_id = m.league_id AND subquery.home_team_api_id = m.home_team_api_id
GROUP BY
    m.season,
    l.name
ORDER BY
    m.season,
    l.name;

-- Output (first 10 rows):
season   |league                  |best_team               |max_wins|
---------+------------------------+------------------------+--------+
2008/2009|Belgium Jupiler League  |Tubize                  |      15|
2008/2009|England Premier League  |Wigan Athletic          |      16|
2008/2009|France Ligue 1          |Valenciennes FC         |      14|
2008/2009|Germany 1. Bundesliga   |VfL Wolfsburg           |      16|
2008/2009|Italy Serie A           |Udinese                 |      14|
2008/2009|Netherlands Eredivisie  |Willem II               |      14|
2008/2009|Poland Ekstraklasa      |Śląsk Wrocław           |      13|
2008/2009|Portugal Liga ZON Sagres|Vitória Setúbal         |      11|
2008/2009|Scotland Premier League |St. Mirren              |      15|
2008/2009|Spain LIGA BBVA         |Villarreal CF           |      14|


--6. Which players are the tallest and which are the shortest?
SELECT
    "name",
    height,
    category
FROM
    (SELECT
        player_name AS "name",
        height,
        CASE
            WHEN rank() OVER (ORDER BY height DESC) <= 10 THEN 'Tallest'
            WHEN rank() OVER (ORDER BY height ASC) <= 10 THEN 'Shortest'
        END AS category
    FROM Player p) AS subquery
WHERE
    category IS NOT NULL
ORDER BY height DESC;

-- Output:
name                  |height|category|
----------------------+------+--------+
Kristof van Hout      |208.28|Tallest |
Bogdan Milic          | 203.2|Tallest |
Costel Pantilimon     | 203.2|Tallest |
Fejsal Mulic          | 203.2|Tallest |
Jurgen Wevers         | 203.2|Tallest |
Kevin Vink            | 203.2|Tallest |
Lacina Traore         | 203.2|Tallest |
Nikola Zigic          | 203.2|Tallest |
Paolo Acerbis         | 203.2|Tallest |
Pietro Marino         | 203.2|Tallest |
Stefan Maierhofer     | 203.2|Tallest |
Vanja Milinkovic-Savic| 203.2|Tallest |
Zeljko Kalac          | 203.2|Tallest |
Anthony Deroin        |162.56|Shortest|
Bakari Kone           |162.56|Shortest|
Edgar Salli           |162.56|Shortest|
Fouad Rachid          |162.56|Shortest|
Frederic Sammaritano  |162.56|Shortest|
Lorenzo Insigne       |162.56|Shortest|
Pablo Piatti          |162.56|Shortest|
Quentin Othon         |162.56|Shortest|
Samuel Asamoah        |162.56|Shortest|
Diego Buonanotte      |160.02|Shortest|
Maxi Moralez          |160.02|Shortest|
Juan Quero            |157.48|Shortest|


--7. What is the average height and weight of a soccer player?
SELECT
	AVG(height) AS "avg height",
	AVG(weight) AS "avg weight"
FROM Player p;

-- Output:
avg height        |avg weight        |
------------------+------------------+
181.86744484628662|168.38028933092224|


--8. What is the average overall rating of the players?
SELECT 
    AVG(sub.overall_rating) AS "AVG overall rating"
FROM
    (SELECT 
        overall_rating
    FROM
        Player_Attributes
    WHERE
        (player_api_id, date) IN
            (SELECT 
                player_api_id, MAX(date)
            FROM
                Player_Attributes
            GROUP BY
                player_api_id)) AS sub;

-- Output:
AVG overall rating|
------------------+
 67.96500904159132|


--9. Which players have the greatest potential in the 2015/2016 season?
SELECT DISTINCT 
	(SELECT 
		player_name 
	FROM Player p 
	WHERE p.player_api_id = pa.player_api_id) AS "player name",
	potential
FROM Player_Attributes pa
WHERE 
	date > "2016-01-01"
ORDER BY potential DESC
LIMIT 10;

-- Output:
player name      |potential|
-----------------+---------+
Neymar           |       94|
James Rodriguez  |       93|
Paul Pogba       |       91|
Alen Halilovic   |       90|
Antoine Griezmann|       90|
David De Gea     |       90|
Eden Hazard      |       90|
Manuel Neuer     |       90|
Youri Tielemans  |       90|
Antoine Griezmann|       89|


--10. Which players have the best overall rating?
SELECT 
    p.player_name AS "player name",
    pa.overall_rating AS "player rating"
FROM
    (SELECT 
        player_api_id,
        overall_rating
    FROM
        Player_Attributes
    WHERE
        (player_api_id, date) IN
            (SELECT 
                player_api_id, MAX(date)
            FROM
                Player_Attributes
            WHERE
                date BETWEEN '2015-01-01' AND '2016-12-31'
            GROUP BY
                player_api_id)
        ORDER BY overall_rating DESC
        LIMIT 10) AS pa
JOIN Player p ON pa.player_api_id = p.player_api_id;

-- Output:
player name       |player rating|
------------------+-------------+
Lionel Messi      |           94|
Cristiano Ronaldo |           93|
Luis Suarez       |           90|
Manuel Neuer      |           90|
Neymar            |           90|
Arjen Robben      |           89|
Zlatan Ibrahimovic|           89|
Andres Iniesta    |           88|
Eden Hazard       |           88|
Mesut Oezil       |           88|


--------------------------------------------------------------------------------------
-- After those questions, I found two interesting topics to dig into.
--------------------------------------------------------------------------------------

--1. If player's height has an impact on their overall rating or potential?
--------------------------------------------------------------------------------------

-- Average player's height:
SELECT
	AVG(height) AS "avg height"
FROM Player p;

--Output:
avg height        |
------------------+
181.86744484628662|

-- Average overall rating and potential for all players:
SELECT 
    COUNT(sub.player_api_id) AS "Count",
    AVG(sub.potential) AS "AVG potential",
    AVG(sub.overall_rating) AS "AVG overall rating"
FROM
    (SELECT 
        player_api_id,
        potential,
        overall_rating
    FROM
        Player_Attributes
    WHERE
        (player_api_id, date) IN
            (SELECT 
                player_api_id, MAX(date)
            FROM
                Player_Attributes
            GROUP BY
                player_api_id)) AS sub;

--Output:
Count|AVG potential    |AVG overall rating|
-----+-----------------+------------------+
11064|71.04005424954792| 67.96500904159132|

-- Average overall rating and potential for players with height above 181 cm:
SELECT 
    COUNT(pa.player_api_id) AS "Count",
    AVG(pa.potential) AS "AVG potential",
    AVG(pa.overall_rating) AS "AVG overall rating"
FROM
    (SELECT 
        player_api_id,
        potential,
        overall_rating
    FROM
        Player_Attributes 
    WHERE
        (player_api_id, date) IN
            (SELECT 
                player_api_id, MAX(date)
            FROM
                Player_Attributes 
            GROUP BY
                player_api_id)) AS pa
LEFT JOIN Player p  on pa.player_api_id = p.player_api_id
WHERE height > 181;

--Output:
Count|AVG potential    |AVG overall rating|
-----+-----------------+------------------+
 5873|70.89676320272572| 68.07495741056218|

-- Average overall rating and potential for players with height below 181 cm:
SELECT 
    COUNT(pa.player_api_id) AS "Count",
    AVG(pa.potential) AS "AVG potential",
    AVG(pa.overall_rating) AS "AVG overall rating"
FROM
    (SELECT 
        player_api_id,
        potential,
        overall_rating
    FROM
        Player_Attributes 
    WHERE
        (player_api_id, date) IN
            (SELECT 
                player_api_id, MAX(date)
            FROM
                Player_Attributes 
            GROUP BY
                player_api_id)) AS pa
LEFT JOIN Player p  ON pa.player_api_id = p.player_api_id
WHERE height < 181;

--Output:
Count|AVG potential    |AVG overall rating|
-----+-----------------+------------------+
 5191|71.20211946050097| 67.84065510597303|

--FINDINGS
-- Player's height has no significant impact on their overall rating or potential.
-- There are 13% more players with height above average than below.



--2. What skills are better taller players in? And what skills are better shorter players in?
--------------------------------------------------------------------------------------

-- Average skills ratings for players above 181 cm:
WITH sub AS (
    SELECT 
        player_api_id,
        potential,
        overall_rating,
        crossing,
        finishing,
        heading_accuracy,
        short_passing,
        volleys,
        dribbling,
        curve,
        free_kick_accuracy,
        long_passing,
        ball_control,
        acceleration,
        sprint_speed,
        agility,
        reactions,
        balance,
        shot_power,
        jumping,
        stamina,
        strength,
        long_shots,
        aggression,
        interceptions,
        positioning,
        vision,
        penalties,
        marking,
        standing_tackle
    FROM
        Player_Attributes 
    WHERE
        (player_api_id, date) IN (
            SELECT 
                player_api_id,
                MAX(date)
            FROM
                Player_Attributes 
            GROUP BY
                player_api_id
        )
)
SELECT 
    COUNT(sub.player_api_id) AS "Count",
    AVG(sub.potential) AS "AVG potential",
    AVG(sub.overall_rating) AS "AVG overall rating",
    AVG(sub.crossing) AS "AVG crossing",
    AVG(sub.finishing) AS "AVG finishing",
    AVG(sub.heading_accuracy) AS "AVG heading accuracy",
    AVG(sub.short_passing) AS "AVG short passing",
    AVG(sub.volleys) AS "AVG volleys",
    AVG(sub.dribbling) AS "AVG dribbling",
    AVG(sub.curve) AS "AVG curve",
    AVG(sub.free_kick_accuracy) AS "AVG free kick accuracy",
    AVG(sub.long_passing) AS "AVG long passing",
    AVG(sub.ball_control) AS "AVG ball control",
    AVG(sub.acceleration) AS "AVG acceleration",
    AVG(sub.sprint_speed) AS "AVG sprint speed",
    AVG(sub.agility) AS "AVG agility",
    AVG(sub.reactions) AS "AVG reactions",
    AVG(sub.balance) AS "AVG balance",
    AVG(sub.shot_power) AS "AVG shot power",
    AVG(sub.jumping) AS "AVG jumping",
    AVG(sub.stamina) AS "AVG stamina",
    AVG(sub.strength) AS "AVG strength",
    AVG(sub.long_shots) AS "AVG long shots",
    AVG(sub.aggression) AS "AVG aggression",
    AVG(sub.interceptions) AS "AVG interceptions",
    AVG(sub.positioning) AS "AVG positioning",
    AVG(sub.vision) AS "AVG vision",
    AVG(sub.penalties) AS "AVG penalties",
    AVG(sub.marking) AS "AVG marking",
    AVG(sub.standing_tackle) AS "AVG standing tackle"
FROM
    sub
LEFT JOIN Player ON sub.player_api_id = Player.player_api_id
WHERE
    Player.height > 181;

-- Output:
Count|AVG potential    |AVG overall rating|AVG crossing      |AVG finishing    |AVG heading accuracy|AVG short passing|AVG volleys      |AVG dribbling    |AVG curve        |AVG free kick accuracy|AVG long passing |AVG ball control  |AVG acceleration|AVG sprint speed |AVG agility      |AVG reactions    |AVG balance      |AVG shot power   |AVG jumping      |AVG stamina       |AVG strength     |AVG long shots   |AVG aggression    |AVG interceptions|AVG positioning   |AVG vision       |AVG penalties    |AVG marking      |AVG standing tackle|
-----+-----------------+------------------+------------------+-----------------+--------------------+-----------------+-----------------+-----------------+-----------------+----------------------+-----------------+------------------+----------------+-----------------+-----------------+-----------------+-----------------+-----------------+-----------------+------------------+-----------------+-----------------+------------------+-----------------+------------------+-----------------+-----------------+-----------------+-------------------+
 5873|70.89676320272572| 68.07495741056218|46.971039182282794|43.19318568994889|   58.39216354344123|57.73696763202726|42.88153928001419|50.67495741056218|45.07075722645859|     42.42010221465077|52.59011925042589|56.955536626916526|58.8526405451448|60.34923339011925|57.28267423302004|64.96166950596252|55.40432700833481|57.57189097103918|66.20766093278951|61.777683134582624|73.55519591141397|46.61959114139693|61.214310051107326|50.79437819420784|47.208177172061326|51.61500266004611|49.70936967632027|47.53390119250426|  50.64599659284497|


-- Average skills ratings for players below 181 cm:
WITH sub AS (
    SELECT 
        player_api_id,
        potential,
        overall_rating,
        crossing,
        finishing,
        heading_accuracy,
        short_passing,
        volleys,
        dribbling,
        curve,
        free_kick_accuracy,
        long_passing,
        ball_control,
        acceleration,
        sprint_speed,
        agility,
        reactions,
        balance,
        shot_power,
        jumping,
        stamina,
        strength,
        long_shots,
        aggression,
        interceptions,
        positioning,
        vision,
        penalties,
        marking,
        standing_tackle
    FROM
        Player_Attributes 
    WHERE
        (player_api_id, date) IN (
            SELECT 
                player_api_id,
                MAX(date)
            FROM
                Player_Attributes 
            GROUP BY
                player_api_id
        )
)
SELECT 
    COUNT(sub.player_api_id) AS "Count",
    AVG(sub.potential) AS "AVG potential",
    AVG(sub.overall_rating) AS "AVG overall rating",
    AVG(sub.crossing) AS "AVG crossing",
    AVG(sub.finishing) AS "AVG finishing",
    AVG(sub.heading_accuracy) AS "AVG heading accuracy",
    AVG(sub.short_passing) AS "AVG short passing",
    AVG(sub.volleys) AS "AVG volleys",
    AVG(sub.dribbling) AS "AVG dribbling",
    AVG(sub.curve) AS "AVG curve",
    AVG(sub.free_kick_accuracy) AS "AVG free kick accuracy",
    AVG(sub.long_passing) AS "AVG long passing",
    AVG(sub.ball_control) AS "AVG ball control",
    AVG(sub.acceleration) AS "AVG acceleration",
    AVG(sub.sprint_speed) AS "AVG sprint speed",
    AVG(sub.agility) AS "AVG agility",
    AVG(sub.reactions) AS "AVG reactions",
    AVG(sub.balance) AS "AVG balance",
    AVG(sub.shot_power) AS "AVG shot power",
    AVG(sub.jumping) AS "AVG jumping",
    AVG(sub.stamina) AS "AVG stamina",
    AVG(sub.strength) AS "AVG strength",
    AVG(sub.long_shots) AS "AVG long shots",
    AVG(sub.aggression) AS "AVG aggression",
    AVG(sub.interceptions) AS "AVG interceptions",
    AVG(sub.positioning) AS "AVG positioning",
    AVG(sub.vision) AS "AVG vision",
    AVG(sub.penalties) AS "AVG penalties",
    AVG(sub.marking) AS "AVG marking",
    AVG(sub.standing_tackle) AS "AVG standing tackle"
FROM
    sub
LEFT JOIN Player ON sub.player_api_id = Player.player_api_id
WHERE
    Player.height < 181;

-- Output:
Count|AVG potential    |AVG overall rating|AVG crossing      |AVG finishing    |AVG heading accuracy|AVG short passing|AVG volleys       |AVG dribbling    |AVG curve         |AVG free kick accuracy|AVG long passing  |AVG ball control|AVG acceleration |AVG sprint speed |AVG agility      |AVG reactions    |AVG balance     |AVG shot power   |AVG jumping     |AVG stamina      |AVG strength    |AVG long shots   |AVG aggression    |AVG interceptions|AVG positioning  |AVG vision       |AVG penalties  |AVG marking      |AVG standing tackle|
-----+-----------------+------------------+------------------+-----------------+--------------------+-----------------+------------------+-----------------+------------------+----------------------+------------------+----------------+-----------------+-----------------+-----------------+-----------------+----------------+-----------------+----------------+-----------------+----------------+-----------------+------------------+-----------------+-----------------+-----------------+---------------+-----------------+-------------------+
 5191|71.20211946050097| 67.84065510597303|61.898265895953756|53.79460500963391|  54.988631984585744|66.36088631984586|53.453166093465505|65.74296724470135|59.663766943151934|     54.72023121387283|60.570134874759155|67.9102119460501|71.45105973025048|70.56473988439306|72.04551891563828|65.94219653179191|73.2033178231843|64.74393063583815|66.6075257940522|67.79633911368015|62.1393063583815|58.17514450867052|60.775915221579965|51.32986512524085|60.29749518304432|61.61521343313777|57.673795761079|46.55741811175337|  50.43236994219653|

-- FINDINGS
-- Players below average height in average have better ratings in particular skills. Their best skills are balance, agility, and acceleration.
-- Players above average height's best skills are strength, jumping, and reaction.
-- However, those are only average results. Those numbers define players' results in the season 2015/2016.
-- The Dataset is too small to assume that height is a key player performance factor.

