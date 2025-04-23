# League of Legends FF at 15?
## Introduction
League of Legends is a team-based strategy game where two teams of five powerful champions face off to destroy the other’s base. Choose from over 140 champions to make epic plays, secure kills, and take down towers as you battle your way to victory. [^1]

In League of Legends, each match features two teams—one on the blue side and one on the red side—each composed of five unique champions filling 5 unique roles. The objective is to destroy the enemy team’s Nexus before they destroy yours.

The game map consists of three main lanes (top, mid, and bot) connected by jungle areas filled with neutral monsters. Each lane is protected by three layers of turrets (towers) that must be destroyed in sequence to reach the opposing base.

During the early game—typically the first 15 minutes—teams compete for key objectives that can provide a significant advantage. These include securing dragons that grant team-wide buffs, Void Grubs that assist in destroying towers, and First Blood and First Tower, which awards bonus gold to the first player to score a kill and the first team that takes down a turret. Winning these early skirmishes can set the pace for the rest of the match.

In this project, I will be looking at in-game statistics during the first 15 minutes of a game to determine if a team should FF (surrender) the game.[^2]

The dataset I will be working with is from Oracle's Elixir's 2024 match data which contains thousands of pro-play matches and their statistics. The dataset has `117648` rows and `161` columns. Each match has `12` rows in the csv, one row for each player on each team and one row for each team.

The relevant columns are:
`result`
: The result of the game. `0` for a loss, `1` for a win.

`side`
: Which side the team is on. `Red` or `Blue`.

`firstblood`
: If a team gets the first kill. `0` for false, `1` for true.

`firstdragon`
: If a team kills the first dragon. `0` for false, `1` for true.

`firsttower`
: If a team kills the first tower. `0` for false, `1` for true.

`turretplates`
: Number of turret plates[^3] a team gets.

`golddiffat15`
: The difference of total gold[^4] between a team and the opposing team at 15 mintutes.

`csdiffat15`
: The difference of total cs[^5] between a team and the opposing team at 15 mintutes.

`xpdiffat15`
: The difference of total xp[^6] between a team and the opposing team at 15 mintutes.

`killsat15`
: The total number of kills a team has at 15 minutes.

-----

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
For this project, I will only be looking entries that are fully complete and the rows of team data.

```py
# number of incomplete rows
league[league['datacompleteness'] != 'complete'].shape 
(16692, 161)
```
Some of the rows in the dataset are not fully complete.
```py
# number of team rows
league[league['position'] == 'team'].shape 
(19608, 161)
```
I only want to use the rows with team data.

```py
# filter out incomplete and non-team data
league_clean = league[(league['datacompleteness'] == 'complete') 
    & (league['position'] == 'team')]
```
Now we only have rows that are complete and are team data. But some are columns are `0`s and `1`s so I will change them into bools for easier reading and analysis.

```py
bool_cols = ['firstblood',
    'firstdragon',
    'firstherald',
    'firstbaron',
    'firsttower',
    'firstmidtower',
    'firsttothreetowers']
league_clean[bool_cols] = league_clean[bool_cols].astype(bool)

league_clean['result'] = league_clean['result'].map(
    {1.0: 'win', 0: 'loss'})
```
The only columns I will be looking at are the ones mentioned above.

```py
league_clean = league_clean[[
    'result',
    'side',
    'firstblood',
    'firstdragon',
    'firsttower',
    'turretplates',
    'golddiffat15',
    'csdiffat15',
    'xpdiffat15',
    'killsat15']]
```

Here's the final cleaned dataframe:
```py
print(league_clean.head().to_markdown(index=False))
```

| result   | side   | firstblood   | firstdragon   | firsttower   |   turretplates |   golddiffat15 |   csdiffat15 |   xpdiffat15 |   killsat15 |
|:---------|:-------|:-------------|:--------------|:-------------|---------------:|---------------:|-------------:|-------------:|------------:|
| win      | Blue   | False        | True          | True         |              8 |           2293 |           23 |          949 |           6 |
| loss     | Red    | True         | False         | False        |              5 |          -2293 |          -23 |         -949 |           3 |
| win      | Blue   | False        | False         | False        |              1 |            -75 |           12 |         1092 |           8 |
| loss     | Red    | True         | True          | True         |              8 |             75 |          -12 |        -1092 |           6 |
| win      | Blue   | False        | True          | False        |              1 |           -561 |          -36 |          410 |           4 |


### Univariate Analysis

<iframe
 src="assets/univariate_kills.html"
 width="800"
 height="416"
 frameborder="0"
 ></iframe>
This histogram shows the distribution of kills at 15 minutes. There seem to be two peaks at 0 and 3, indicating that these are the most common, suggesting that there is not too much action in the early game. Not very many teams achieve 10+ kills in 15 minutes. Maybe the more kills at 15 minutes, the higher the chance of winning the game?

<iframe
 src="assets/univariate_turretplates.html"
 width="800"
 height="416"
 frameborder="0"
 ></iframe>
This histogram shows the distribution of how many turret plates a team destroys. The only peak is at `3`. There might be a correlation between the number of turret plates a team gets and their success later in the game?

### Bivariate Analysis

<iframe
 src="assets/bivariate_killsat15.html"
 width="800"
 height="416"
 frameborder="0"
 ></iframe>

This box plot shows the relationship between the number of kills at 15 minutes and the game's result. There is little separation between the two, with the median only being different by `1`. The first quartiles are also separated by `1`, while the third quartile has a difference of `2`. 

 <iframe
 src="assets/bivariate_turretplates.html"
 width="800"
 height="416"
 frameborder="0"
 ></iframe>

This box plot shows the relationship between number of turret plates and result. There is some difference between the two. For teams who won, the median is `5` while for teams that lost, the median is `3`. The first and third quartiles are also greater for teams who won than teams who lost. 

 <iframe
 src="assets/bivariate_golddiffat15.html"
 width="800"
 height="416"
 frameborder="0"
 ></iframe>

This box plot shows the relationship between gold differential and result. The median for winning teams is `+1377` and is `-1377` for losing teams. This is becasue the distribution is symmetrical. There is a clearer divide here than for kills at 15 or turret plates. This suggests that the higher the gold difference at at 15 minutes, the more likely that team is going to win.

### Interesting Aggregates

```py
(league_clean
 .groupby(['firstblood','firsttower'])
 ['result']
 .value_counts(normalize=True)
 .unstack())
```

| [firstblood] [firsttower] |     loss |      win |
|:--------------------------|---------:|---------:|
| (False, False)            | 0.736003 | 0.263997 |
| (False, True)             | 0.396645 | 0.603355 |
| (True, False)             | 0.603355 | 0.396645 |
| (True, True)              | 0.263997 | 0.736003 |

When grouped by `firstblood` and `firsttower`, I found that when both are true, `73.6%` of the teams won the game. When a team secured the first blood, but not the first tower, the proportion drops down to `39.7%`. If a team gets the first tower but not the first kill, they win `60.3%` of the time. And when a team does not get the first blood or first tower, only `26.4%` of teams end up winning.

```py
league_clean.pivot_table(
    index='firstblood',
    columns='firsttower',
    values='golddiffat15',
    aggfunc='mean' 
)
```

| firsttower    |    False |    True |
| firstblood    |          |         |
|:--------------|---------:|--------:|
| False         | -2411.18 | 785.68  |
| True          | -785.68  | 2411.18 |

Looking at `firstblood` and `firsttower` again, I created a pivot table showing the gold difference at 15. When a team get the first blood and first tower, they have a gold lead of `2411.18`. When a team gets the first blood but not the first tower, they fall behind the other team, with a gold difference of `-785.68`. A team that gets the first tower but not the first blood, the average gold difference is `785.68`. A team that does not get the first blood or the first tower ends up in a huge deficit by 15 minutes, with a gold difference of `-2411.18`. 

Looking at these tables, it seems like destroying the first tower heavily impacts how much gold your team has at 15 minutes and your chances at winning.

### Imputation
```py
league_clean.isna().sum()
```

|              | count |
|:-------------|------:|
| result       |   0   |
| side         |   0   |
| firstblood   |   0   |
| firstdragon  |   0   |
| firsttower   |   0   |
| turretplates |   0   |
| golddiffat15 |   4   |
| csdiffat15   |   4   |
| xpdiffat15   |   4   |
| killsat15    |   4   |


Only 2 games (4 teams) went on for less than 15 minutes, since they did not have statistics for `golddiffat15`, `csdiffat15`, `xpdiffat15`, `killsat15`. It does not make sense to look at these games so I will not do any imputation and just drop these rows.

```py
league_clean = league_clean.dropna()
```

---

## Framing a Prediction Problem

The variable I am trying to predict is the `result` column, which takes on only 2 values: `win` or `loss`. This means that this is a binary classification prediction problem. I chose the `result` column as the response variable because winning is the objective of the game. 

I am trying to predict if a team will win or lose a game after the initial early game phase. Thus, I will only be using statistics from 15 minutes to find an asnwer to my problem. The model I will be using is a logistic regression model. Since I am trying to determine if the game should be forfited, a game prediceted to be a loss, but was actually a win would be very bad. Therefore, I will be evaluating model performance with recall. Models will be trained and tested using a 75%/25% train test split.

## Baseline Model

The baseline model was a logistic regression model that was trained on 2 columns: `turretplates` and `killsat15`. These features were all quantatative, so no feature engineering took place.

After fitting the model, it achieved a training recall score of `0.6014` and a training accuracy of `0.6402`. The testing recall was `0.5915` and testing accuracy was `0.6388`.

This performance is a good starting point, but can be improved upon. Previous analysis of `killsat15` and `turretplates` indicated that the difference between different `result`s could be hard to tell and could possibly explain the suboptimal performance. There is definately room for improvement.

## Final Model

The final model was a logistic regresson model that was trained on the rest of the features, adding `side`, `firstblood`, `firstdragon`, `firsttower`, `golddiffat15`, `csdiffat15`, and `xpdiffat15`.

The columns, `side`, `firstblood`, `firstdragon`, `firsttower` are categorical variables, all of which were one hot encoded by `OneHotEncoder`.

The rest of the columns, `turretplates`, `golddiffat15`, `csdiffat15`, `xpdiffat15`, and `killsat15` are all numeric variables which were all standardized using `StandardScaler`. 

Hyperparameter tuning was conducted using `GridSearchCV` to identify the best `C`, `penalty`, `tol`, and `max_iter` to further improve model performance. The best hyperparameters selected were `C = 0.01`, `max_iter = 500`, `penalty = l1`, and `tol = 0.001`.

With these changes, the final model had a training recall of `0.7462`, a training accuracy of `0.7326`, a testing recall of `0.7563`, and a testing accuracy of `0.7406`, demonstrating a clear improvement upon the baseline model.


<iframe
 src="assets/train_test_acc_recall.html"
 width="800"
 height="416"
 frameborder="0"
 ></iframe>

 As seen in this graph, the final model has significant improvements in recall and accuracy over the baseline model.


[^1]: [League of Legends](https://www.leagueoflegends.com/en-us/how-to-play/)

[^2]: Earliest you can surrender is at 15 minutes.

[^3]: During the first 14 minutes of the game, each turret has 5 "plates" that divide its heath that give damage reduction to the turret. Destroying these plates grants extra gold to the player who destroyed it.

[^4]: Gold is gained from killing minions, enemies, turrets, wards, jungle monsters, etc. Used to buy items from the shop.

[^5]: Creep score; The number of minions and monsters a champion has killed in the game.

[^6]: Gained when a champion is near enemy minions when they die or from kills. Xp allows players to level up their abilities to have stronger effects.

-----
