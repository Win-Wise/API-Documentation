# WinWise Multi-Market arbitrage API Documentation

This document provides an overview of the configuration options for using the WinWise arbitrage betting calculator API. This API will accept your betting data and determine the best wager values such that you will profit no matter the result (a.k.a. arbitrage betting).

# Input Format:

Your input data can be structured similar to the following:
```json
{
    "wager_limit": ...,
    "wager_precision": ...,
    "no-draw": ...,
    "max_process_time": ...,
    "total_max_wager_count": ...,
    "mode": ...,
    "bookmakers": [
        {
            "id": ...,
            "commission": ...,
            "wager_limit": ...,
            "ignore_wager_precision": ...,
            "max_wager_count": ...
        },
        ...
    ],
  * "bets": [
        {
            * "bet_type": ...,
            * "value": ...,
            * "odds": ...,
            "bookmaker": ...,
            "lay": ...,
            "volume": ...,
            "previous_wager": ...
        },
        ...
    ]
}
```
Note: The `*` indicates that this parameter is required.

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

Wager precision was introduced to make betting activity less conspicuous. The `wager_precision` parameter determines the precision of calculated wager amounts. All calculated wager amounts will be a multiple of this value. For instance, setting `wager_precision` to 10 will ensure that all wagers are a multiple of 10, except of course for bets that have an associated bookmaker with `ignore_wager_precision` set to true.

Example:
```json
"wager_precision": 10
```

## No Draw (default: false)

The `no-draw` parameter is a boolean value that indicates whether the event can end in a draw. If this value is set to false the calculator will ensure that your wagers will still be possible if the event ends in a draw. If this value is set to true the calculator will assume that the event cannot end in a draw and will not consider this possibility when calculating wagers. Incorrectly setting this value to false will result in the calculator returning sub optimal profits or no profits at all. Incorrectly setting this value to true will mean that if the game ends in a draw there is no guarantee of any profitable bets.


## Max Process Time (default: 25 seconds)

The `max_process_time` parameter specifies the maximum amount of time the calculator will spend trying to find a solution. If the calculator is unable to find a solution within this time it will return the best solution it has found so far. If this value is greater than 25 it will be reduced to 25. And if the value is lower that the minimum allowed value (this value may change with time) it will be increased to the minimum allowed value, to determine the minimum allowed value set this to 0 (ensure `mode` is 1, default) and measure api response time.

## Total Max Wager Count (default: -1, no limit)

The `total_max_wager_count` parameter specifies the maximum number of wagers that can be placed on the event in total. If no wager limits are defined here or in bookmakers, this value will default to -1 (no limit). If this value is set to -1 and some bookmakers have a 'max_wager_count' value set, the calculator will use the sum of the 'max_wager_count' values as the total max wager count.

## Mode (default: 1)

The `mode` parameter specifies the mode of operation for the calculator. The following modes are available:
    1 - "GuaranteedProfit": The calculator will attempt to find the solution that maximises the minimum profit. This is the default mode.
    2 - "MaxProfit": The calculator will attempt to find the solution that maximises the highest profit. That profit will only apply to one possible outcome of the event, the other outcomes may have a lower profit.


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

The `bets` parameter is an array of `Bet` objects representing information about each available bet. Bet object may have the following properties:

| Required | Property         | Data Type         | Description                                                     | Default Value | Example |
|----------|------------------|-------------------|-----------------------------------------------------------------|---------------|---------|
|   ✔️     | `bet_type`       | Integer or String | The type of bet (see Bet Type Reference).                       |               | 1       |
|   ✔️     | `value`          | String            | Specifics of the bet, formatted for `bet_type`.                 |               | "home"  |
|   ✔️     | `odds`           | Number            | The odds associated with the bet.                               |               | 2.0     |
|          | `bookmaker`      | Integer           | ID of the bookmaker offering this bet.                          | 0             | 1       |
|          | `lay`            | Boolean           | Indicates whether the bet is a 'lay' bet.                       | false         | true    |
|          | `volume`         | Number            | The volume associated with this bet (default: -1 for no limit). | -1.0          | 1000.25 |
|          | `previous_wager` | Number            | Total amount already wagered on this bet at these odds.         | 0.0           | 500.0   |

See [Partially placed bets](#Partially-placed-bets) for more information on how and why to use the `previous_wager` parameter.

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

The API will return data in a similar format to how it was received. In the below formatting specification the `*` indicates that this parameter will be added by the API.
```json
{
    * "profit": ,
    "wager_limit": ...,
    "wager_precision": ...,
    "no-draw": ...,
    "max_process_time": ...,
    "total_max_wager_count": ...,
    "mode": ...,
    "bookmakers": [
        {
            "id": ...,
            "commission": ...,
            "wager_limit": ...,
            "ignore_wager_precision": ...,
            "max_wager_count": ...
        },
        ...
    ],
    "bets": [
        {
          * "wager": ...,
            "bet_type": ...,
            "value": ...,
            "odds": ...,
            "bookmaker": ...,
            "lay": ...,
            "volume": ...,
            "previous_wager": ...
        },
        ...
    ]
}
```

The parameters at the base of the json structure that are returned by the API are only those that were specified by the user and were not the default values.

## Profit:
The `profit` parameter is a list of 2 numbers. e.g. [50.0, 100.0]. The first number is the minimum profit that will be made if the recommended wagers are places. The second number is the maximum profit that could be made should the event have a specific outcome.

## Bets:

Only the bets that have a `wager` parameter or a `previous_wager` parameter will be returned by the API. The `wager` parameter will be set by the API and the `previous_wager` parameter will be set by the user.

### Wager
The `wager` parameter is the calculated wager amount for the bet. This value will be a multiple of the `wager_precision` value, unless the associated bookmaker has `ignore_wager_precision` set to true. This is the value that should be wagered on the bet. If all bets are placed with the recommended wagers the event will result in a profit between the minimum and maximum profit values.

### Partially placed bets:

As you place a bet with a bookmaker you should add the `wager` value to the `previous_wager` value for that bet. This means that in the event that you can't place all the recommended wagers you will be able to send the updated `previous_wager` values to the API and it will return a new set of recommended wagers. 

Note: If you are unable to place a specific bet you should set the bets `volume` parameter to 0.0. If you are unable to place any bets with a specific bookmaker you should either set it's `wager_limit` parameter to 0.0 or set it's `max_wager_count` parameter to 0.0.

## Full example response:

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
