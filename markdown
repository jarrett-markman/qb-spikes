---
title: "Why Quarterback Spikes Need to Change."
output: pdf_document
---
## Abstract
A quarterback spike is a strategy employed by every single NFL team, usually in low-clock scenarios for a variety of purposes. Such as stopping the clock for one final play, whether it be a field-goal attempt or one final offensive play that includes either a “Hail Mary” or a bunch of laterals in an unlikely attempt to score. Other times teams use it to stop the clock for whatever reason, whether it be their players are tired or they feel the clock is getting out of time. Teams prepare for 2-minute drills and late game scenarios, so why are they forced to use a spike to stop the clock and essentially throw an incompletion? 
In almost every end of game situation most NFL teams choose to concede a down and spike the ball to set up the next play. It’s unfair to assume teams will evaluate what to do in late-game situations, however, that got me interested. Why do teams spike the ball anyway? For the most part, a quarterback spike is a purposeful incompletion to stop the clock. In some situations, stopping the clock with a spike is a necessary evil. However, many teams misinterpret the clock and choose to spike the ball, assuming they’ll need more time than necessary.
#### Step 1: Load the libraries.
```{r, results='hide'}
library(tidyverse)
library(ggrepel)
library(ggimage)
library(gt)
library(caret)
library(nflfastR)
options(scipen = 9999)
```
#### Step 2: Create 2 data frames by loading the data from the past 10 years.
```{r}
fourth_quarter_spikes <- load_pbp(2012:2021) %>%
  filter(qb_spike == 1) %>%
  filter(qtr == 4) %>%
  filter(score_differential >=-8)
second_quarter_spikes <- load_pbp(2012:2021) %>%
  filter(qb_spike == 1) %>%
  filter(qtr == 2)
```
#### Step 3: Look at every second and fourth quarter spike in the past 10 years.
```{r, echo=FALSE}
second_quarter_spikes %>%
  ggplot(aes(x=epa, y=wpa)) +
  geom_point(color = 'black', shape = 1) +
  geom_hline(yintercept = mean(second_quarter_spikes$epa), color = 'red') +
  geom_hline(yintercept = 0) +
  geom_vline(xintercept = mean(second_quarter_spikes$wpa), color = 'red') +
  geom_vline(xintercept = 0) +
  labs(x = "EPA",
       y = "WPA",
       title = "EPA and WPA on Second Quarter Spikes",
       caption = "Data: nflfastR") +
  theme(plot.title = element_text(size = 12, face = "bold", hjust = 0.5))
fourth_quarter_spikes %>%
  ggplot(aes(x=epa, y=wpa)) +
  geom_point(color = "black", shape = 1) +
  geom_hline(yintercept = mean(fourth_quarter_spikes$epa), color = 'red') +
  geom_hline(yintercept = 0) +
  geom_vline(xintercept = mean(fourth_quarter_spikes$wpa), color = 'red') +
  geom_vline(xintercept = 0) +
  labs(x = "EPA",
       y = "WPA",
       title = "EPA and WPA on Fourth Quarter Spikes",
       caption = "Data: nflfastR") +
  theme(plot.title = element_text(size = 12, face = "bold", hjust = 0.5))
```
For 2nd quarter and 4th quarter spikes, a lot of the data hovers in the negative EPA and WPA range. The red lines indicate the average EPA and WPA per spike, and are both below 0. Meaning that on an average spike in the past 10 years, it would have a lower win probability and expected points than the play before.
#### Step 4: What variables are important for fourth quarter spikes?
``` {r, echo=FALSE}
fourth_quarter_spikes %>% 
  select(posteam, defteam, time, yrdln, score_differential, epa, wpa, fg_prob) %>%
  filter(score_differential >= -3) %>%
  arrange(time)
fourth_quarter_spikes %>% 
  select(posteam, defteam, time, yrdln, score_differential, epa, wpa, fg_prob) %>%
  filter(score_differential >= -3) %>%
  arrange(-fg_prob)
fourth_quarter_spikes %>%
  select(posteam, defteam, time, yrdln, score_differential, epa, wpa, td_prob) %>%
  filter(score_differential <= -4 & score_differential >= -8) %>%
  arrange(time)
fourth_quarter_spikes %>%
  select(posteam, defteam, time, yrdln, score_differential, epa, wpa, td_prob) %>%
  filter(score_differential <= -4 & score_differential >= -8) %>%
  arrange(-td_prob)
```
In a touchdown score game, both EPA and WPA tend to be negative. 
In a field goal score game WPA tends to be negative, however, EPA seems to be a great indicator of actual end game scenarios, especially in games within 3 points or less. 
##### How many spikes have a negative EPA based on endgame scenarios?
``` {r, echo = FALSE}
fourth_quarter_spikes %>%
  select(epa, score_differential) %>%
  filter(score_differential >= -3) %>%
  filter(epa < 0)
fourth_quarter_spikes %>%
  select(epa, score_differential) %>%
  filter(score_differential <= -4 & score_differential >= -8) %>%
  filter(epa < 0)
```
In field goal score games, there were 83 instances of spikes with a negative EPA, just below 60%.
In touchdown score games, there were 105 instances of spikes with a negative EPA, just below 90%. 
##### What does it look like with a spike to set up a field goal to win the game?
``` {r, echo = FALSE}
fourth_quarter_spikes %>%
  select(posteam, defteam, posteam_score, defteam_score, time, yrdln, epa, fg_prob, score_differential) %>%
  filter(score_differential > -3) %>%
  arrange(-fg_prob)
```
With a high field goal probability, less than 3 point deficit and little time remaining, EPA tends to be positive.
This makes a lot of practical sense in game-time scenarios. If you can, it makes perfect sense to spike the ball - or throw an incompletion - before a field goal to set up a chance to win the game. 
#### Step 6: Are certain teams better with using quarterback spikes?
The data can be joined with the built in function teams_colors_logos from nflfastR to plot the data with team logos.
``` {r, echo = FALSE}
fourth_quarter_spikes <- fourth_quarter_spikes %>%
  left_join(teams_colors_logos, by = c("posteam" = "team_abbr"))
second_quarter_spikes <- second_quarter_spikes %>%
  left_join(teams_colors_logos, by = c("posteam" = "team_abbr"))
```
```{r, echo=FALSE}
fourth_quarter_spikes %>%
  ggplot(aes(x=epa, y=wpa)) +
  geom_image(aes(image = team_logo_espn), size = .05) +
  geom_hline(yintercept = 0) +
  geom_vline(xintercept = 0) +
  labs(x = "Fourth Quarter Spike EPA",
       y = "Fourth Quarter Spike WPA",
       title = "EPA and WPA for Fourth Quarter Spikes",
       caption = "Data: nflfastR") +
  theme(plot.title = element_text(size = 12, face = "bold", hjust = 0.5))
second_quarter_spikes %>%
  ggplot(aes(x=epa, y=wpa)) +
  geom_image(aes(image = team_logo_espn), size = .05) +
  geom_hline(yintercept = 0) +
  geom_vline(xintercept = 0) +
  labs(x = "Second Quarter Spike EPA",
       y = "Second Quarter Spike WPA",
       title = "EPA and WPA for Second Quarter Spikes",
       caption = "Data: nflfastR") +
  theme(plot.title = element_text(size = 12, face = "bold", hjust = 0.5))
```
##### Which teams have the highest EPA and WPA per spike?
``` {r, echo = FALSE}
fourth_quarter_spikes %>%
  select(posteam, epa, wpa) %>%
  group_by(posteam) %>%
  summarise(avg_epa = mean(epa), avg_wpa = mean(wpa)) %>%
  arrange(-avg_epa)
```
The Eagles, Chiefs, Vikings and 49ers are some of the most efficient teams when spiking the ball in late game scenarios looking at expected points and win probability added.
##### Now teams can be plotted with their average EPA and WPA.
``` {r, echo=FALSE}
fourth_quarter_spikes %>%
  select(posteam, epa, wpa) %>%
  group_by(posteam) %>%
  summarise(avg_epa = mean(epa), avg_wpa = mean(wpa)) %>%
  left_join(teams_colors_logos, by = c('posteam' = 'team_abbr')) %>%
  ggplot(aes(x=avg_epa, y=avg_wpa)) +
  geom_image(aes(image = team_logo_espn), size = 0.05) +
  labs(x= "EPA",
       y = "WPA",
       title = "Average Fourth Quarter Spike EPA and WPA by team",
       caption = "Data: nflfastR") +
  theme(plot.title = element_text(size = 10, face = "bold", hjust = 0.5)) +
  geom_segment(aes(x = mean(avg_epa), y = mean(avg_wpa), xend = mean(avg_epa), yend = 0.01)) +
  geom_segment(aes(x = mean(avg_epa), y = mean(avg_wpa), xend = 0.5, yend = mean(avg_wpa))) +
  geom_segment(aes(x = mean(avg_epa), y = mean(avg_wpa), xend = mean(avg_epa), yend = -0.05)) +
  geom_segment(aes(x = mean(avg_epa), y = mean(avg_wpa), xend = -0.3, yend = mean(avg_wpa))) 
```
As stated, the Eagles, 49ers, Vikings and Chiefs all seem to have some kind of advantage in decision-making when they spike the ball. 
#### Step 7: Is there a way to make a multivariate regression model to predict the EPA of a spike?
I want to create 2 different models, one for field goal score games, and one for touchdown score games. 
First there need to be 2 more data frames for a fg_model and a td_model.
``` {r}
fg_model <- fourth_quarter_spikes %>%
  filter(score_differential >= -3) %>%
  select(epa, game_seconds_remaining, down, yardline_100, posteam_timeouts_remaining, score_differential, fg_prob)
td_model <- fourth_quarter_spikes %>%
  filter(score_differential <= -4) %>%
  select(epa, game_seconds_remaining, down, yardline_100, posteam_timeouts_remaining, score_differential, td_prob)
```
Now I can create a model.
``` {r}
fg_model <- lm(epa ~ ., data = fg_model)
td_model <- lm(epa ~ ., data = td_model)
```
# Here are the results!
#### Field Goal Situation
``` {r, echo = FALSE}
summary(fg_model)
```
#### Touchdown Situation
``` {r, echo = FALSE}
summary(td_model)
```
In order to spike the ball beneficially in a field goal score game, a team but have good field position and field goal probability. It makes sense for there to be negative coefficients for seconds remaining and the down because the margin for error grows larger with more time and a later down. Additionally, if a team has at least 1 timeout, there is really no need to spike the ball. 
The predictive touchdown EPA is definitely flawed, especially considering that touchdown probability has a negative coefficient attached to it, meaning that the more likely a touchdown is the less sense a spike makes. With all of the EPA data for touchdown score games being so negative the model is really flawed because it doesn't associate touchdown probability with the team being more likely to score.
## Conclusion
More than half the time, a quarterback spike has a negative EPA and WPA. There are only a select few of scenarios in which a spike becomes acceptable in terms of EPA or WPA. A spike has the same effect of an incompletion. While only 1 second may have been taken off the clock instead of 5, the chance of a successful play is removed because with a spike, there is one possible outcome. Similar to a quarterback kneel, there is only 1 goal in the play. While the goal is accomplished, most of the time it has a negative effect on the overall outcome. Spiking the ball on 1st and goal with 30 seconds left only wastes a down. The odds that time runs out is so low, especially considering that you are going for the end zone, which will most likely result in a touchdown or incompletion. Spiking the ball on first down won't change the overall goal of going for the end zone. the difference between a loss of 1 second versus 5 is smaller than a purposeful incompletion versus an attempt at more yardage or a touchdown. Many teams misjudge the clock, a spike with 30+ seconds doesn't really help a teams chances to win the game. If teams prepare for 2 minute drills, why do they choose to enforce spikes. Shouldn't they be prepared for these scenarios. The only time that a spike should really happen is at the end of a game when a team is setting up to kick a field goal to win or tie the game. Otherwise, it's not sensible to spike the ball for the sole purpose of spiking the clock. That's the sole purpose of a spike, to stop the clock. Most often, the quarterback fails to realize the surrounding variables of the situation, such as the possibility of offensive success, time manipulation or even if there are timeouts remaining. By sacrificing a down to a spike, you lose the possibility of better yardage, down, and a chance to score. All a spike is, is an incompletion that takes 1 second rather than 5+. That is why spikes need to change. Teams should start to spike the ball less, unless absolutely necessary, such as to set up a field goal to tie or win the game. Otherwise, spikes should begin to become obsolete. 