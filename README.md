# League of Legends Win/Loss Classification

Author: Alexander Takamoto

## Introducton
### General Introduction

League of Legends is a 5v5 Multiplayer Online Battler Arena game by Riot Games. It is one of the most popular esports in the world with a very large fanbase and thousands of professional matches being played each year. In addition to the esports side, millions of people play League of Legends casually or for a rank and it is one of the biggest online games in general. 

One of the common areas of discussion in ranked play whether forfeiting is the best option or if it's still possible to win the game at a certain point. The earliest possible point to forfeit the game is at 15 minutes and this leads to the term *"ff15"* being used. Within games it is widely debated if you should play out games even if you are losing. People either feel that you should forfeit to save people's time or you should play it out because you could comeback. This project investigates this question at the professional level.

More clearly:

*Can you determine who will win in the first 15 minutes of a professional League of Legends game, without using any prior knowledge of team or player history?*

### Dataset Overview

The dataset used in this analysis comes from Oracle's Elixir and contains data for all of the League of Legends professional matches played in 2022. 

The dataset contains 150588 rows, each pertaining to either a player or a team as a whole, and 161 columns. Each game has 12 rows: 10 for individual players and 2 for the overall team performances.

Of the 161 columns, some of the main columns of interest are as follows:

- `gameid`: An identifier for each individual match.

- `league`: The league or tournament the game was played in

- `side`: The team color of the team or player. Either red or blue.

- `position`: The role the player played during the game. If it is a team row, the value is 'team'

- `kills`: The amount of eliminations of the opponent champions. There are also columns for kills at different times of the game.

- `deaths`: The amount of times eliminated during the game.

- `assists`: The amount of times contributed to a kill without getting the kill.

- `result`: Whether the team or player won or lost. 1 is a win and 0 is a loss.

While I will not use the columns for kills, deaths, and assists directly, there are columns for these at specific timestamps which I will use instead. The specific timestamps are 10, 15, 20, and 25. Here the 15 minute timestamp will be most useful.

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning

First, I focused on removing columns that you would not have access to at certain points of the game. This includes columns like the general `kills`, `deaths`, and `assists` columns as well as columns like `dragons`, `inhibitors`, and basically any column that has end of game or late game results.

After that I worked on restructuring the dataset. The original dataset as mentioned above, has 12 rows per game. I adjusted this so that there were only two rows per game (one for each team). The player rows were moved to columns based on their `position`. This means that columns such as `killsat15_mid` and `csat20_top` were added.

Continuing on I realized that in some leagues there is no data for the columns I would need to answer my question. These include: 

- *ASCI:*               0% of data is complete

- *DCup:*              0% of data is complete

- *LDL:*                0% of data is complete

- *LPL:*               0% of data is complete

- *WLDs:*               91% of data is complete

After some research, ASCI and DCup are Chinese tournaments and LDL and LPL are the two main Chinese leagues. The data missing from WLDs are from Chinese qualifier games. After consideration, since there is no data for these leagues in the revelant columns, I will drop the all rows corresponding to these leagues and the rows with missing values from Worlds. This means that these results cannot be generalized to Chinese games. 

Finally, I created new features focused on the differences in kills, deaths, assists, cs, xp and so on for each timestamp, generally and for each specific role. I also dropped one side from each game so there is only one row per game. The opposing team's specific stats still remain in columns such as `opp_killsat10`.

This is the head of the newly cleaned dataframe.

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameid</th>
      <th>league</th>
      <th>result</th>
      <th>goldat10</th>
      <th>xpat10</th>
      <th>csat10</th>
      <th>opp_goldat10</th>
      <th>opp_xpat10</th>
      <th>opp_csat10</th>
      <th>golddiffat10</th>
      <th>...</th>
      <th>killsdiffat20</th>
      <th>killsdiffat25</th>
      <th>assistsdiffat10</th>
      <th>assistsdiffat15</th>
      <th>assistsdiffat20</th>
      <th>assistsdiffat25</th>
      <th>deathsdiffat10</th>
      <th>deathsdiffat15</th>
      <th>deathsdiffat20</th>
      <th>deathsdiffat25</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ESPORTSTMNT01_2690210</td>
      <td>LCKC</td>
      <td>0</td>
      <td>16218.0</td>
      <td>18213.0</td>
      <td>322.0</td>
      <td>14695.0</td>
      <td>18076.0</td>
      <td>330.0</td>
      <td>1523.0</td>
      <td>...</td>
      <td>-2.0</td>
      <td>-1.0</td>
      <td>5.0</td>
      <td>-8.0</td>
      <td>-12.0</td>
      <td>-10.0</td>
      <td>-3.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ESPORTSTMNT01_2690219</td>
      <td>LCKC</td>
      <td>0</td>
      <td>14939.0</td>
      <td>17462.0</td>
      <td>317.0</td>
      <td>16558.0</td>
      <td>19048.0</td>
      <td>344.0</td>
      <td>-1619.0</td>
      <td>...</td>
      <td>-4.0</td>
      <td>-7.0</td>
      <td>-2.0</td>
      <td>-2.0</td>
      <td>-5.0</td>
      <td>-12.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ESPORTSTMNT01_2690227</td>
      <td>LCKC</td>
      <td>1</td>
      <td>15466.0</td>
      <td>19600.0</td>
      <td>368.0</td>
      <td>15569.0</td>
      <td>18787.0</td>
      <td>355.0</td>
      <td>-103.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>-1.0</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>11.0</td>
      <td>1.0</td>
      <td>-2.0</td>
      <td>-2.0</td>
      <td>-3.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>ESPORTSTMNT01_2690255</td>
      <td>LCKC</td>
      <td>0</td>
      <td>15978.0</td>
      <td>17714.0</td>
      <td>297.0</td>
      <td>15641.0</td>
      <td>18229.0</td>
      <td>334.0</td>
      <td>337.0</td>
      <td>...</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>9.0</td>
      <td>-2.0</td>
      <td>-2.0</td>
      <td>-4.0</td>
      <td>-3.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>ESPORTSTMNT01_2690264</td>
      <td>LCKC</td>
      <td>1</td>
      <td>15346.0</td>
      <td>18131.0</td>
      <td>331.0</td>
      <td>16247.0</td>
      <td>19128.0</td>
      <td>344.0</td>
      <td>-901.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-2.0</td>
      <td>4.0</td>
      <td>7.0</td>
      <td>12.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 450 columns</p>
</div>

There are still many columns that will not be used for the model but that will be explored during the analysis.

### Univariate Analysis

<iframe
  src="assets/GoldDiffHist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This distribution appears to be approximately normal and centered roughly around 0 which would mean that the teams are even in gold at 10 minutes. This makes sense and since there is no or very little skew, it means there is a balance of teams with gold leads or teams behind in gold at 10 minutes. 

<iframe
  src="assets/TeamKillsHist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This distribution is skewed to the right which makes since since it is not possible to have negative team kills. Based on the histogram it appears that the majority of the games have between 0 and 4 kills with anything above 8 being an outlier.

### Bivariate Analysis

<iframe
  src="assets/PosCSPie.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This pie chart shows the results of games when you have a positive CS difference at 10 minutes. Given that about 65% of the time teams win when we there is a positive CS difference at 10 minutes we can get a general idea of how important columns like `csat10` or `csat15` are.

### Interesting Aggregates

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>golddiffat10_mid</th>
      <th>golddiffat15_mid</th>
      <th>golddiffat20_mid</th>
      <th>xpdiffat10_mid</th>
      <th>xpdiffat15_mid</th>
      <th>xpdiffat20_mid</th>
      <th>killsdiffat10_mid</th>
      <th>killsdiffat15_mid</th>
      <th>killsdiffat20_mid</th>
    </tr>
    <tr>
      <th>result</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-132.325069</td>
      <td>-332.630460</td>
      <td>-603.252170</td>
      <td>-85.591303</td>
      <td>-222.370917</td>
      <td>-454.404893</td>
      <td>-0.165683</td>
      <td>-0.382723</td>
      <td>-0.730268</td>
    </tr>
    <tr>
      <th>1</th>
      <td>139.121636</td>
      <td>375.036957</td>
      <td>667.332613</td>
      <td>74.294761</td>
      <td>217.180301</td>
      <td>416.643784</td>
      <td>0.184966</td>
      <td>0.414424</td>
      <td>0.750270</td>
    </tr>
  </tbody>
</table>
</div>

By grouping by result and looking at the midlane columns, we can see the average stats for midlaners who win and loss the game. This gives a general idea of where we would expect the midlaners stat line to be around at different points in the game if their team were to win or lose the game.

## Assessment of Missingness
## NMAR Analysis

When looking at the general dataset before the cleaning, the column `playername` may be Not Missing At Random (NMAR), because players whose names that are more well known may be less likely to be missing. In order to make a column like this Missing at Random (MAR) a column like the number of games the player was a part of may be helpful or a column if the player was a substitute or something along those lines.

## Missingness Dependency

Even after removing the rows from the games played in China and cleaning the dataset, there are still some missing values in the stats for the different minute timestamps. In this section I will test whether the column `killsat25` depends `poscsdiffat10` or `league`. For both tests, the significance level will be 0.05 and the test testistic will be total variation distance (TVD).

First I perform a permutation test about the missingness of `killsat25` depending on `league`.

*Null Hypothesis:* The distribution of `league` when `killsat25` isn't missing is the same as the distribution of `league` when `killsat25` is missing.

*Alternative Hypothesis:* The distribution of `league` when `killsat25` isn't missing is the NOT the same as the distribution of `league` when `killsat25` is missing.

This is the observed distribution of `league` when `killsat25` is missing and not missing.

<iframe
  src="assets/LeagueDist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Below is a histogram illustrating the results of the permutation test.

<iframe
  src="assets/LeagueTVD.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

After running the permutation test, the *p-value* of the observed test statistic was 0.00. Since the p-value of 0.00 is less than the 0.05 signicance level, we *reject the null hypothesis* and convlude that the missingness of `killsat25` likely depends on `league`.

The next permutation test is about the missingness of `killsat25` depending on `poscsdiffat10`.

*Null Hypothesis:* The distribution of `poscsdiffat10` when `killsat25` isn't missing is the same as the distribution of `poscsdiffat10` when `killsat25` is missing.

*Alternative Hypothesis:* The distribution of `poscsdiffat10` when `killsat25` isn't missing is the NOT the same as the distribution of `poscsdiffat10` when `killsat25` is missing.

This is the observed distribution of `poscsdiffat10` when `killsat25` is missing and not missing.

<iframe
  src="assets/CSDist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Below is a histogram illustrating the results of the permutation test.

<iframe
  src="assets/CSTVD.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

After running the permutation test, the *p-value* of the observed test statistic was 0.16. Since the p-value of 0.16 is greater than the 0.05 signicance level, we *fail to reject the null hypothesis* and it is not likely that the missingness of `killsat25` depends on `poscsdiffat10`.

## Hypothesis Testing

In this section, I will determine if there is a significant difference in the distributions of gold difference at 15 minutes. This will allow us to gauge how team differences like gold vary across winning and losing teams and if that difference is significant. I will be performing a permutation test at a 0.05 significance level using absolute mean difference as a test statistic.

*Null Hypothesis:* The distribution of gold differences at 15 minutes for the winning team is the same as the distribution for the losing team.

*Alternative Hypothesis:* The distribution of gold differences at 15 minutes for the winning team is NOT the same as the distribution for the losing team.

Below is a histogram illustrating the results of the permutation test.

<iframe
  src="assets/GoldTVD.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

After running the permutation test, the p-value of the observed test statistic was 0.00. Since the *p-value* is 0.00, which is less than the 0.05 signicance level, we *reject the null hypothesis* and conlude that it is likely that the distribution of gold differences at 15 minutes for the winning team is different from the distribution for the losing team.

## Framing a Prediction Problem

*The goal of the model is to determine whether a team will win or lose a professional League of Legends game only based on game data available at 15 minutes.* Thus this model will not use information from later points like `killsat25` or general game information that you would only know after the game like the general column `assists`. It will also not use information about the specific teams or players involved. This is a binary classification problem with the response variable being `result` which is either a 1 if the team won or 0 if the team lost.

In this model I am choosing to focus on accuracy as the primary metric to evaluate the model. There is no inherent reason to prioritize false positives(incorrectly predicting a win) or false negatives(incorrectly predicting a loss). Also to note, when I looked at the classification report the precision and recall scores were similar for both classes, meaning that it doesn't not appear that the model disproportionately favors one class over the other.

## Baseline Model

For the baseline model, I used a Random Forest Classifier with 6 features. The first 3 features were `killsdiffat15`, `assistsdiffat15`, and `deathsdiffat15`. Additionally, I applied a binarization transformation to `killsdiffat10`, `assistsdiffat10`, and `deathsdiffat10`, converting them into binary values indicating whether the difference was positive or negative. This transformation helps the model capture whether a team had an advantage or disadvantage in these key statistics at 10 minutes and gives a general idea of how the game was going at this point in time.

After fitting the model, the accuracy score was *0.679*. While this accuracy is decent, looking back at the bivariate analysis, 65.4% of teams that have a `poscsdiffat10` win the game. Given that just based on this one feature earlier in the game, around 65% of teams win, I hope to improve this accuracy by adding more features and adjusting hyperparameters.

## Final Model

In the final model, I added 8 more features. `csdiffat15` and `xpdiffat15` provide more information about how the laning phase (early game) is going for the team as a whole. The xp and cs columns generally show if team members are winning lanes. The other 6 columns are the general `golddiffat15` along with the role specific gold values: `golddiffat15_top`, `golddiffat15_jng`, `golddiffat15_mid`, `golddiffat15_bot`, and `golddiffat15_sup`. This is to give a general idea of how each lane is doing since some lanes may provide more importance than others. For example, a Support having a good game may not have as much importance as a Jungler or Mid Laner. Therefore while, general differences are important, it is also useful to give a general idea of how each lane is doing.

In addition to this I used GridSearchCV and ended up with the hyperparameters: max_depth = 5 and n_estimators = 200.

After these adjustments and additions, the accuracy score is now *0.757*. This is a nice improvement from the baseline model and suggests that adding these additional roles specific and laning features helped to model better differentiate between winning and losing teams.

## Fairness Analysis

Now I will look at two different groups as determine if the model is fair among the two groups. To do this group X will be teams who have a gold lead at 15 minutes while group Y will be teams who are behind in gold at 15 minutes. I will be using difference in accuracy as the test statistic at a 0.05 significance level.

*Null Hypothesis:* The accuracy for teams with a gold lead at 15 minutes is the same for teams behind in gold at 15 minutes.

*Alternative Hypothesis:* The accuracy for teams with a gold lead at 15 minutes is NOT the same for teams behind in gold at 15 minutes.

After performing the test, the *p-value* was 0.746. Since the p-value is greater than 0.05, we *fail to reject the null hypothesis* and thus it seems that our model predicts results from each of the two groups at a similar accuracy.