# SQL-fifa2023

**Problem 1: List all players that are unavailable due to disciplinary reasons (i.e 2 yellow cards or 1 red card).**

SQL QUERY: 
```
    select reciever, card_color, COUNT(cards.id) as count, players.first_name, players.last_name
    from cards
    inner join players 
    on players.id = cards.reciever
    group by card_color, reciever
    having card_color = "Red" or card_color = "Yellow" and count = 2;
```


**Hur har vi strukturerat tabellerna för att klara av detta?**
Svar: Vi har en tabell med spelare (players) och en tabell med kort (cards) som i en kolumn hänvisar till player_id (vi har döpt kolumnen till "reciever"). 

**Problem 2: List a team’s matches and results.**
SQL QUERY: 
````
    select games.id, home_team_id, away_team_id, winner_id, stage, teams."name" from games
    inner join teams on teams.id = games.winner_id
    where home_team_id = 14 or away_team_id = 14
````



**Problem 3: List a teams roster with players and coach, goals, assists, shots and disciplinary, matches played, matches started, minutes played. Number of clean sheets and save percentage for the goalkeepers**

SQL QUERY: 
````
select DISTINCT(players.first_name), coaches.first_name as coach_name, goals.scorer_id, goals.conceder_id, goals.assist_id, players.shots, cards.card_color, players_games.subbed_in, players.minutes_played
from players
inner join coaches on coaches.team_id = players.team_id
left join goals on goals.scorer_id = players.id
left join cards on cards.reciever = players.id
left join players_games on players_games.player_id = players.id
where players.team_id = 14;
````

**Problem 4: List the playoff tree with team abbreviations and -flags, score (if any)/date and time if no result.**

SQL QUERY: 
````
SELECT games.*, goals.*, teams.name, teams.abbreviation from games 
inner join schedule on schedule.game_id = games.id
inner join teams on games.home_team_id = teams.id
inner join goals on goals.game_id = games.id
where games.id > 49
````

**Problem 5: List the top-10 players sorted first by goals, then by assists**
SQL QUERY: 
````
SELECT first_name, last_name, SUM(n_goals) as n_goals, SUM(n_assists) as n_assists
FROM(
	SELECT players.id, players.first_name as first_name, players.last_name as last_name, count(players.id) as n_goals, 0 as n_assists
	FROM goals
	LEFT JOIN players
	ON players.id = goals.scorer_id
	GROUP BY players.id
	
	UNION
	
	SELECT players.id, players.first_name as first_name, players.last_name as last_name, 0 as n_goals, count(players.id) as n_assists
	FROM goals
	LEFT JOIN players
	ON players.id = goals.assist_id
	GROUP BY players.id
)
GROUP BY id
ORDER BY n_goals DESC, n_assists DESC LIMIT 10;
````

**Problem 6: Detailed info for a finished game including teams, players, goals, disciplinary, substitutions, referee, venue, date. Every situation often includes one or more players, a time and sometimes additional info.**
SQL QUERY: 
````
	SELECT distinct(players.first_name), players.last_name, players.team_id,
	players_games.player_id, goals.*, players_games.game_id, schedule.date, players_games.subbed_in,
	H.name as home_team, A.name as away_team, games.venue_id, venues.name, games.winner_id, games.stage, games.head_referee_id,
	referees.first_name, referees.last_name, cards.card_color
	from players

	INNER join teams
	on players.team_id = teams.id
	LEFT join players_games
	on players.id = players_games.player_id
	inner join games
	on games.id = players_games.game_id
	inner join referees
	on games.head_referee_id = referees.id
	inner join teams H
	on H.id = games.home_team_id
	inner join teams A
	on A.id = games.away_team_id
	inner join schedule
	on schedule.game_id = games.id
	inner join venues
	on venues.id = games.venue_id
	left join cards
	on players_games.player_id = cards.reciever
	LEFT join goals
	on goals.scorer_id = players_games.player_id
	WHERE stage like "final"

	order by players.first_name ASC;
````

**Problem 7: List all games today.**
SQL QUERY: 
````
select schedule.*, games.*, H.name as home_team, A.name as away_team  from schedule
inner join games 
on schedule.game_id = games.id
inner join teams H
on H.id = games.home_team_id
inner join teams A
on A.id = games.away_team_id
where date = "2023-07-22"
````

**Problem 8: List a group table with teams, wins, draws, losses, goal difference and points.**
SQL QUERY: 
````
SELECT teams.*, scored_goals - conceded_goals as goal_difference
FROM teams
INNER JOIN(
	SELECT id as team_id, SUM(conceded) as conceded_goals, SUM(scored) as scored_goals
	FROM(
		SELECT teams.id, 0 as conceded, count(teams.id) as scored
	    FROM goals
	    INNER JOIN players
	    ON goals.scorer_id = players.id
	    INNER JOIN teams
	    ON players.team_id = teams.id
	    WHERE teams.group_name = "A"
	    GROUP BY teams.id
	
	    UNION
	
	    SELECT teams.id, count(teams.id) as conceded, 0 as scored
	    FROM goals
	    INNER JOIN players
	    ON goals.conceder_id = players.id
	    INNER JOIN teams
	    ON players.team_id = teams.id
	    WHERE teams.group_name = "A"
	    GROUP BY teams.id
	)
	GROUP BY id
)
ON team_id = teams.id;
````
