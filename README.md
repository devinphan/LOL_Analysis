# Vision to Victory: Predicting League of Legends Match Outcomes Through Early Game Analytics
A project for DSC80 at UCSD.

## **Introduction**

This project utilizes the 2022 League of Legends Esports Match Dataset from Oracle's Elixir, containing comprehensive statistics from professional League of Legends competitions during the 2022 Spring split. League of Legends is one of the world's most popular esports, with professional matches watched by millions globally and featuring multi-million dollar prize pools. Understanding what leads to victory in these high-stakes competitions is not just a matter of gaming interest—it's a serious analytical challenge with implications for team strategy, player development, and even sports betting markets.

## **Dataset**

This project utilizes the **2022 League of Legends Esports Match Dataset** from Oracle's Elixir, containing professional LoL match statistics from the 2022 Spring split. The dataset includes player and team-level statistics for games across multiple global regions.

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| **gameid** | String | Unique identifier for each match. |
| **datacompleteness** | String | Indicates if data for the match is 'complete' or 'partial'. |
| **league** | String | The tournament region/league (e.g., LCK, LPL, NLC). |
| **date** | DateTime | The date and time the match started. |
| **result** | Integer | The outcome of the match from the row's perspective (1 = Win, 0 = Loss). This is the **target variable** for prediction. |
| **position** | String | The in-game role for player rows (top, jng, mid, bot, sup). This column is null for team-level summary rows. |
| **champion** | String | The champion selected by the player. |
| **gamelength** | Integer | The duration of the match in seconds. |
| **golddiffat15** | Numeric | **A key feature.** The gold difference between the teams at the 15-minute mark. |
| **killsat15** | Numeric | The team's total kills at the 15-minute mark. |
| **firstblood** | Boolean | Indicates which team secured the first kill of the match. |
| **firstdragon** | Boolean | Indicates which team secured the first Dragon objective. |
| **visionscore** | Numeric | A score representing the team's vision control (wards placed, destroyed, etc.). |
