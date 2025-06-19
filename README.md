# UMich Women's Volleyball Analysis

#### Author: Christian Fiuczynski

## Introduction

This dataset captures play-by-play data from a University of Michigan women's volleyball match against Indiana University, Bloomington, collected via DataVolley, totaling 1,616 rows of in game actions such as serves, digs, blocks, and attacks. The question I try to answer is if we can predict whether a player's attack will be successful based on contextual features in the game. The relevant column names are `team`, `opponent`, `player_name`, `skill`, `evaluation_code`, `set_number`, `rally_number`, `video_time`, `score`, `point_won_by`, `home_score`, `away_score`, `red_zone`, and `efficiency`.
The `player_name` column identifies the athlete performing the action. `set_number` and `rally_number` provide temporal context, indicating when in the match the action occurred. `red_zone` is a binary variable indicating whether both teams had more than 20 points and the difference was within 2 points. Lastly, `efficiency` is translated column from `evaluation code` that quantifies attack outcomes: 1 for a kill, 0 for a neutral result, and -1 for an error or blocked attack.

## Data Cleaning and Exploratory Data Analysis

This dataset was originally a `.dvw` file as opposed to a standard `.csv`. An example of this file pre-parsed looks like this:

---
[3SET]
True;2-8;6-16;8-21;14-25;25;
True;4-8;8-16;15-21;21-25;25;
True;4-8;14-16;18-21;20-25;25;
True;;;;;25;
True;;;;;15;
[3PLAYERS-H]
0;1;1;;;*;;;-416101;Williams;Rachel;Williams;;;False;;;
0;2;2;3;3;1;;;-416098;Chicoine;Chloe;Chicoine;;;False;;;
0;3;3;*;*;6;;;-478520;McAleer;Ryan;McAleer;;;False;;;
0;4;4;2;2;*;;;-416099;Wollard;Kenna;Wollard;;;False;;;
0;5;5;5;5;3;;;-426784;Anderson;Taylor;Anderson;;;False;;;
0;6;6;;;;;;-490950;Foster;Sienna;Foster;;;False;;;
0;7;7;4;4;2;;;-332942;Colvin;Raven;Colvin;;;False;;;
0;8;8;;;;;;-510312;Gray;Raven;Gray;;;False;;;
0;9;9;1;1;5;;;-385034;Meyers;Lourdes;Meyers;;;False;;;
0;10;10;*;*;*;;;-332941;Hornung;Ali;Hornung;;;False;;;
0;11;11;;;;;;-479694;Shondell;Allie;Shondell;;;False;;;
0;14;12;;;;;;-426783;Heaney;Grace;Heaney;;;False;;;
---

From the datavolley library, there is a `read_cv` function responsible for reading `.dvw` files. Once the file has been read, the dataframe can be created by calling the `.get_plays()` method. 

| team                            | player_name    | skill     | evaluation_code   |   set_number |   rally_number |   home_team_score |   visiting_team_score | red_zone   |   efficiency |
|:--------------------------------|:---------------|:----------|:------------------|-------------:|---------------:|------------------:|----------------------:|:-----------|-------------:|
| Indiana University, Bloomington | Camryn Haworth | Serve     | -                 |            1 |              1 |                 1 |                     0 | False      |            0 |
| University of Michigan          | Maddi Cuchran  | Reception | +                 |            1 |              1 |                 1 |                     0 | False      |            0 |
| University of Michigan          | Morgan Burke   | Set       | #                 |            1 |              1 |                 1 |                     0 | False      |            1 |
| University of Michigan          | Amalia Simmons | Attack    | -                 |            1 |              1 |                 1 |                     0 | False      |            0 |
| Indiana University, Bloomington | Madi Sell      | Block     | +                 |            1 |              1 |                 1 |                     0 | False      |            0 |
| Indiana University, Bloomington | Ramsey Gary    | Dig       | #                 |            1 |              1 |                 1 |                     0 | False      |            1 |
| Indiana University, Bloomington | Camryn Haworth | Set       | #                 |            1 |              1 |                 1 |                     0 | False      |            1 |
| Indiana University, Bloomington | Avry Tatum     | Attack    | #                 |            1 |              1 |                 1 |                     0 | False      |            1 |
| University of Michigan          | Morgan Burke   | Dig       | =                 |            1 |              1 |                 1 |                     0 | False      |           -1 |

## Univariate Analysis

I first selected to investigate the `efficiency` column that I am trying to predict. The plot below shows that most Umich attacks are rated with 0 efficiency, indicating neutral outcomes (e.g., attacks that don’t immediately result in a point or error). Positive outcomes (1, kills) are more common than negative ones (-1, blocks or errors), suggesting an overall balanced but slightly offense-leaning dataset. This trend supports our prediction goal by highlighting the class distribution and potential need for balanced classification techniques.

<iframe
  src="assets/dist-atk-eff.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Bivariate Analysis

I also plotted how both teams attack efficiency varies across sets, with a comparison between red zone and non-red zone rallies. It reveals a drop in efficiency during red zone moments in later sets, highlighting how game pressure impacts performance, providing an insight directly relevant to our goal of predicting success under match conditions.

<iframe
  src="assets/red-zone-eff.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Interesting Aggregates

This pivot table compares each Michigan player's average attack efficiency in red zone versus non-red zone situations. It reveals how performance varies under pressure—some players maintain consistent efficiency, while others see noticeable drops or improvements in red zone contexts. These insights help identify which players are most reliable during critical scoring moments.

<iframe
  src="assets/pivot-red-zone.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
