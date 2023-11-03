# WinWise Multi-Market arbitrage API Documentation

This document provides an overview of the configuration options for using the WinWise arbitrage betting calculator API. This API will accept your betting data and determine the best wager values such that you will profit no matter the result (a.k.a. arbitrage betting).

Your input data should be structured similar to the following:
```json
{
    "wager_limit": ...,
    "wager_precision": ...,
    "bookmakers": [ ... ],
    "bets": [ ... ]
}
```

You can customize the behaviour of the API using the following parameters:

## Wager Limit (default: -1, no wager limit)

The `wager_limit` parameter specifies the maximum amount that can be wagered on the event in total. If no wager limits are defined here or in bookmakers, this value will default to 100, such that all wager values represent a percentage of the total stake.

Example:
```json
"wager_limit": 1000
```

### Note:
When placing lay bets the calculator will ensure the liability of the bet does not exceed any wager limits. This will guarantee that you are never expose more than your wager limit.

## Wager Precision (default: 0.01)

Wager precision was introduced to make betting activity less conspicuous. The `wager_precision` parameter determines the precision of calculated wager amounts. All calculated wager amounts will be a multiple of this value. For instance, setting `wager_precision` to 10 will ensure that all calculated wagers are a multiple of 10, except of course for bets that have an associated bookmaker with `ignore_wager_precision` set to true.

Example:
```json
"wager_precision": 10
```

## Bookmakers

The `bookmakers` parameter allows you to define an array of bookmakers with their specific settings. Each bookmaker object may have the following properties:

| Required | Property                | Data Type  | Description                                                      | Default Value |
|----------|-------------------------|------------|------------------------------------------------------------------|---------------|
|   ✔️     | `id`                    | Integer    | Unique identifier of the bookmaker.                              |               |
|          | `commission`            | Number     | The commission rate charged by the bookmaker.                    | 0.0           |
|          | `wager_limit`           | Number     | The maximum amount that can be wagered with this bookmaker.      | -1 (no limit) |
|          | `ignore_wager_precision`| Boolean    | Ignore the `wager_precision` value for all bets within bookmaker | `false`       |
|          | `max_wager_count`       | Integer    | Maximum number of wagers allowed.                                | -1 (no limit) |


Example:
```json
"bookmakers": [
    {
        "id": 1,
        "commission": 0.05,
        "wager_limit": 500,
        "ignore_wager_precision": true
    },
    {
        "id": 2,
        "max_wager_count": 2
    }
]
```

### Notes:
- It is generally advised to set `ignore_wager_precision` to true for bookmakers that are unlikely to ban you for suspicious betting activity. This will allow the calculator to optimise your wagers to be more profitable.
- If no bookmakers are defined, the Calculator will default to using a single bookmaker with an ID of 0 and no commission or wager limit.

## *Bets

The `bets` parameter is an array of betting options for the event. Bet object may have the following properties:

| Required | Property         | Data Type         | Description                                                     | Default Value | Example |
|----------|------------------|-------------------|-----------------------------------------------------------------|---------------|---------|
|   ✔️     | `bet_type`       | Integer or String | The type of bet (see Bet Type Reference).                       |               | 1       |
|   ✔️     | `value`          | String            | Specifics of the bet, formatted for `bet_type`.                 |               | "home"  |
|   ✔️     | `odds`           | Number            | The odds associated with the bet.                               |               | 2.0     |
|          | `bookmaker`      | Integer           | ID of the bookmaker offering this bet.                          | 0             | 1       |
|          | `lay`            | Boolean           | Indicates whether the bet is a 'lay' bet.                       | false         | true    |
|          | `volume`         | Number            | The volume associated with this bet (default: -1 for no limit). | -1.0          | 1000.25 |
|          | `previous_wager` | Number            | Total amount already wagered on this bet at these odds.         | 0.0           | 500.0   |
|          | `wager`          | Number            | Calculated wager to place on this bet (set by calculator).      | 0.0           | 250.0   |


Example:
```json
"bets": [
    {
        "bet_type": 1,
        "value": "home",
        "odds": 2.0,
        "bookmaker": 1
    },
    {
        "bet_type": 1,
        "value": "home -0.75",
        "odds": 1.8,
        "bookmaker": 2,
        "lay": true
    }
]
```
# Table of Bet Types and Values:
|`bet_type` ID|`bet_type` name         |`value` regex pattern                            |`value` examples                                                              |`bet_type` description                                                                                                                                                                                                                                                 |
|-----------|----------------------|-----------------------------------------------|----------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|1          |MatchWinner           |`^(home\|draw\|away)$`                             |'home', 'draw', 'away'                                                      |The result of the match                                                                                                                                                                                                                                              |
|2          |AsianHandicap         |`^(home\|away) ([+-]?\d+(?:\.(?:0\|25\|5\|75))?)$`   |'home -0.75', 'away -3.0', 'home 4.25', 'away 1.75'                         |Pays if your team/bet wins with a handicap applied, also pays intermediate amount if the result is close for some handicap values. For an in depth demonstration of asian handicap payouts see https://bet-ibc.com/betting-tools/asian-handicap-and-overunder-calculator/|
|3          |Goals_OverUnder       |`^(over\|under) ([+-]?\d+(?:\.(?:0\|25\|5\|75))?)$`  |'over 0.75', 'under 3.0', 'over 4.25', 'under 1.75'                         | ^^                                                                                                                                                                                                                                                                  |
|4          |BothTeamsToScore      |`^(yes\|no)$`                                     |'yes', 'no'                                                                 |A bet on if both teams score                                                                                                                                                                                                                                         |
|5          |ExactScore            |`^([0-9]{1,4}):([0-9]{1,4})$`                    |'2:1'                                                                       |A bet on the final score of the game formatted as Home_Team_Score:Away_Team_Score                                                                                                                                                                                    |
|6          |DoubleChance          |`^(home\|draw\|away)\/(home\|draw\|away)$`           |'home/draw', 'home/away', 'draw/home', 'draw/away', 'away/home', 'away/draw'|A bet that pays out if either result occurs                                                                                                                                                                                                                          |
|7          |Team_OverUnder        |`^(home\|away) (over\|under) ([0-9]{1,4}).5$`      |'home over 2.5', 'away under 0.5'                                           |A bet that the final score for a team will be over or under a number. The number must have a decimal of .5.                                                                                                                                                          |
|8          |OddEven               |`^(odd\|even)$`                                   |'odd', 'even'                                                               |A bet on if the sum of both teams scores will be even or odd.                                                                                                                                                                                                        |
|9          |Team_OddEven          |`^(home\|away) (odd\|even)$`                       |’home odd', 'home even', 'away odd', 'away even'                            |A bet on if a specified teams final score is odd or even.                                                                                                                                                                                                            |
|10         |Result_BothTeamsScore |`^(home\|draw\|away)\/(yes\|no)$`                   |'home/yes', 'draw/yes', 'away/yes', 'home/no', 'draw/no', 'away/no'         |A bet on the result of the game combined with a bet on if both teams will have scored points. Only pays if both bets would pay.                                                                                                                                      |
|11         |Result_TotalGoals     |`^(home\|draw\|away)\/(over\|under) ([0-9]{1,4}).5$`|'home/over 2.5', 'draw/under 3.5', 'away/over 4.5', 'draw/under 0.5'        |A bet on the result of the game and a bet on the total number of goals scored. Only pays if both bets would pay.                                                                                                                                                     |
|12         |TeamCleanSheet        |`^(home\|away) (yes\|no)$`                         |’home yes', ‘away yes’, ‘home no’, 'away no'                                |A bet that your selected team will not allow it’s opposition to score a point.                                                                                                                                                                                       |
|13         |Team_WinToNil         |`^(home\|away) (yes\|no)$`                         |’home yes', ‘away yes’, ‘home no’, 'away no'                                |A bet that your selected team will not allow it’s opposition to score a point and that your selected team will score a point thus winning.                                                                                                                           |
|14         |ExactGoalsNumber      |`^([0-9]{1,4})$`                                 |'2', '3', '4', ‘400’                                                        |A bet on the total number of points scored at the end of the game.                                                                                                                                                                                                   |
|15         |Team_ExactGoalsNumber |`^(home\|away) ([0-9]{1,4})$`                     |'home 2', 'away 3'                                                          |A bet on the points scored by your selected team.                                                                                                                                                                                                                    |
|16         |Team_ScoreAGoal       |`^(home\|away) (yes\|no)$`                         |’home yes', ‘away yes’, ‘home no’, 'away no'                                |A bet that your selected team will or will not score a goal.                                                                                                                                                                                                         |

# Return Format:
The API will return a JSON object with the same structure as the input. Each element in the `bets` array will have had the `wager` element updated to a number reflecting what you should wager on that bet. The `wager` value will remain as 0.0 if you should not place that wager. Also a `profit` value will be added to the object, that is your minimum guaranteed profit from this arbitrage.

Example:
```json
{
    "wager_limit": 1000,
    "wager_precision": 10,
    "bookmakers": [
        {
            "id": 1,
            "commission": 0.05,
            "wager_limit": 500,
            "ignore_wager_precision": true
        },
        {
            "id": 2,
            "max_wager_count": 2
        }
    ],
    "bets": [
        {
            "bet_type": 1,
            "value": "home",
            "odds": 2.0,
            "bookmaker": 1,
            "wager": 250.0
        },
        {
            "bet_type": 1,
            "value": "home -0.75",
            "odds": 1.8,
            "bookmaker": 2,
            "lay": true,
            "wager": 250.0
        }
    ],
    "profit": 50.0
}
```
