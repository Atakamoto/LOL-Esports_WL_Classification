# League of Legends Win/Loss Classification

UCSD DSC80 Project

Author: Alexander Takamoto

## Introducton
### General Introduction

League of Legends is a 5 vs 5 multiplayer online batter area game by Riot Games. It one of the most popular esports in the world, with a very large esports scene and thousands of professional matches being played each year. In addition to the esports side, millions of people play League of Legends casually or for a rank and it is one of the biggest games in general. 

One of the common areas of discussion in ranked play is the idea of forfeiting and if it is still possible to win the game at a certain point. The earliest possible point to forfeit the game is at 15 minutes and this leads to the term "ff15" being used. Within games it is widely debated if you should play out games even if you are losing. People either feel that you should forfeit to save people's time or you should play it out because you could comeback. This project investigates this question at the professional level.

More clearly:

Can you determine who will win in the first 15 minutes of a professional League of Legends game, without using any prior knowledge of team or player history?

### Dataset Overview

The dataset used in this analysis comes from Oracle's Elixir and contains data for all of the League of Legends professional matches played in 2022. 

The dataset contains 150588 rows, each pertaining to either a player or a team as a whole, and 161 columns. For each game there are 12 rows, as there are 10 for all 10 players and 2 for each of the teams involved in the game.

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

- ASCI               0% of data is complete

- DCup               0% of data is complete

- LDL                0% of data is complete

- LPL                0% of data is complete

- WLDs               91% of data is complete

After some research, ASCI and DCup are Chinese tournaments and LDL and LPL are the two main Chinese leagues. The data missing from WLDs are from Chinese qualifier games. After consideration, since there is no data for these leagues in the revelant columns, I will drop the values for these leagues and the missing values from Worlds. This means that these results cannot be generalized to Chinese games. 

Finally, I created new features focused on the differences in kills, deaths, assists, cs, xp and so on for each timestamp, generally and for each specific role. I also dropped one side from each game so there is only one row per game. The other side's specific stats still remain as there are columns for stats like `opp_killsat10`.

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