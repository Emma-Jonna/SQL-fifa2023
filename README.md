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
Svar: Vi har en tabell med spelare (players) och en tabell med kort (cards) som i en kolumn hänvisar till spelar_id.



**Problem 2: List a team’s matches and results.**

SQL QUERY:

**Hur?**
Svar:

**Problem 3: