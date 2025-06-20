# Umich Women's Volleyball Analysis

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

From the datavolley library, there is a read_dv function responsible for reading .dvw files. Once the file has been read, the dataframe can be created by calling the .get_plays() method. These two steps together handle the majority of the data cleaning process internally. The read_dv function parses the structured text file format used by DataVolley, extracting key sections such as player rosters, set scores, and rally-level details. When .get_plays() is called, it returns a tidy pandas DataFrame where each row represents a single in-game action (e.g., Serve, Set, Attack), with relevant metadata such as player_name, skill, evaluation_code, set_number, rally_number, and team scores already decoded and aligned. This function also maps internal shorthand codes to human-readable labels and filters out non-play rows or metadata, meaning that minimal manual cleaning is required. As a result, the dataset is ready for immediate exploration and modeling without significant preprocessing, making the analysis pipeline much more efficient.

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

I first selected to investigate the `efficiency` column that I am trying to predict. The plot below shows that most Umich attacks are rated with 0 efficiency, indicating neutral outcomes (such as attacks that don’t immediately result in a point or error). Positive outcomes (1, kills) are more common than negative ones (-1, blocks or errors), suggesting an overall balanced but slightly offense-leaning dataset. This trend supports our prediction goal by highlighting the class distribution and potential need for balanced classification techniques.

<iframe
  src="assets/dist-atk-eff.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

## Bivariate Analysis

I also plotted how both teams attack efficiency varies across sets, with a comparison between red zone and non-red zone rallies. It reveals a drop in efficiency during red zone moments in later sets, highlighting how game pressure impacts performance, providing an insight directly relevant to our goal of predicting success under match conditions.

<iframe
  src="assets/red-zone-eff.html"
  width="800"
  height="400"
  frameborder="0"
></iframe>

## Interesting Aggregates

This pivot table compares each Michigan player's average attack efficiency in red zone versus non-red zone situations. It reveals how performance varies under pressure—some players maintain consistent efficiency, while others see noticeable drops or improvements in red zone contexts. These insights help identify which players are most reliable during critical scoring moments. Players without red zone efficiencies means they were not in the game at that time or, if they were, did not attack.

<iframe
  src="assets/pivot-red-zone.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Framing a Prediction Problem

The goal of this project is to predict the success of an attack during a volleyball rally, based on contextual features like player identity, set number, and court zone position. This is a multiclass classification problem because the response variable, `efficiency`, has three possible values: 1.0 (successful attack or kill), 0.0 (neutral or returned), and -1.0 (error or blocked). These values represent different levels of effectiveness, making it important to distinguish between more than two categories.

I chose `efficiency` as the response variable because it is a core measure of player and team performance in volleyball. Predicting attack efficiency allows coaches and analysts to identify patterns that lead to successful plays, optimize strategies, and evaluate individual player tendencies.

To evaluate model performance, I used the F1-score. This metric balances both precision and recall, and it accounts for class imbalance by weighting each class according to its frequency. Compared to accuracy, the F1-score provides a more nuanced assessment in cases where certain outcomes (like errors or kills) are less common but highly important.

## Baseline Model

My baseline model is a multiclass classification model using a RandomForestClassifier to predict efficiency, which takes on values of -1.0, 0.0, or 1.0, corresponding to errors, neutral plays, and successful attacks respectively. The model uses four features: set_number (ordinal), rally_number (quantitative), red_zone (binary/nominal), and player_name (nominal). I encoded the nominal player_name feature using OneHotEncoder, while the remaining numerical features were passed through without scaling, as Random Forest models are not sensitive to feature scaling.

The model achieved an accuracy of **54%**, with a macro average F1-score of 0.44. While this performance is modest, it's reasonable given the class imbalance and small dataset. The model struggles particularly with predicting the -1.0 class, but performs better for 0.0 and 1.0, suggesting it may capture some useful patterns for neutral and successful plays. There's  room for improvement through additional feature engineering or balancing techniques.

### Classification Report for Baseline Model

              precision    recall  f1-score   support

        -1.0       0.20      0.15      0.17        39
         0.0       0.64      0.64      0.64       178
         1.0       0.49      0.52      0.50       107

    accuracy                           0.54       324
    macro avg      0.44      0.44      0.44       324
    weighted avg   0.53      0.54      0.54       324

## Final Model

In the final model, I incorporated four features: `player_name`, `set_number`, `rally_number`, and `red_zone`, selected based on their potential relevance to predicting attack outcome efficiency in volleyball. `player_name` helps capture individual performance tendencies and styles, while set_number reflects game progression, which may influence fatigue or strategic shifts. rally_number accounts for the flow and length of a point, which could correlate with complex play development, and red_zone identifies whether the play occurred in a high-pressure scoring situation. These variables collectively encode a mix of player identity, game phase, and situational context, offering a well-rounded basis for classification.

For the modeling algorithm, I used a RandomForestClassifier with class_weight='balanced' to mitigate class imbalance. To enhance model generalization, I implemented a preprocessing pipeline that applied OneHotEncoder to `player_name`, StandardScaler to `rally_number`, and QuantileTransformer to `set_number`. Hyperparameters were tuned using GridSearchCV with 5 fold cross-validation, testing combinations of tree depth and number of estimators. The best performing configuration used n_estimators=100 and max_depth=10, achieving a weighted F1-score of approximately 0.59, an improvement over the baseline model, which used fewer features and lacked feature transformation. This improvement in accuracy suggests that the engineered features and tuning choices better capture patterns in the data that relate to attack efficiency.

### Classification Report for Final Model

              precision    recall  f1-score   support

        -1.0       0.15      0.26      0.19        31
         0.0       0.73      0.69      0.71       200
         1.0       0.56      0.51      0.53        93

    accuracy                           0.59       324
    macro avg      0.48      0.48      0.48       324
    weighted avg   0.63      0.59      0.61       324

#### Confusion Matrix for Final Model

<iframe
  src="assets/final-conf-mat.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Conclusion
From this exploration of the Michigan Women’s Volleyball attack data, I found that predicting the efficiency of an offensive play is a challenging task, but not without meaningful insights. When building a model to classify attack efficiency using features such as player_name, set_number, rally_number, and red_zone, our final model achieved a weighted F1-score of approximately **0.59**. Given the imbalance and limited sample size in some efficiency classes (especially low-efficiency attacks), this result is promising. The classification task proved difficult due to subtle differences in the input features and limited observable patterns in rally dynamics alone. However, the modeling process revealed that certain players and set contexts—like late set rallies or red zone conditions—can influence attack outcomes. For future research, incorporating spatial data (such as attack location or block positioning) or richer contextual features (like pass quality or opponent positioning) could offer stronger predictive signals. It would also be beneficial to utilize a larger dataset. Overall, this project illustrates the complexity of performance analytics in volleyball and highlights the value of structured data in driving tactical insight.









