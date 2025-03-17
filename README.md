Here is where I explain the purpose of the project

Collecting the Data:
```
library(slider)
library(nflfastR)
library(dplyr, warn.conflicts = FALSE)
library(gsisdecoder)
library(car)
ids <- nflfastR::fast_scraper_schedules(2017:2019) %>%
  dplyr::filter(game_type == "SB") %>%
  dplyr::pull(game_id)
pbp <- nflfastR::build_nflfastR_pbp(ids)
```
Loading the Play-by-Play Data
```
data_2024 <- load_pbp(2024)
  data_2024$season <- 2024
data_2023 <- load_pbp(2023)
  data_2023$season <- 2023
data_2022 <- load_pbp(2022)
  data_2022$season <- 2022
data_2021 <- load_pbp(2021)
  data_2021$season <- 2021
data_2020 <- load_pbp(2020)
  data_2020$season <- 2020
data_2019 <- load_pbp(2019)
  data_2019$season <- 2019
data_2018 <- load_pbp(2018)
  data_2018$season <- 2018
data_2017 <- load_pbp(2017)
  data_2017$season <- 2017


pbp_data <- rbind(data_2024,data_2023,data_2022,data_2021,data_2020,data_2019,data_2018,data_2017)
#View(pbp_data)
```
Creating the Schedule
```
schedule_data_2024 <- fast_scraper_schedules(2024)
schedule_data_2023 <- fast_scraper_schedules(2023)
schedule_data_2022 <- fast_scraper_schedules(2022)
schedule_data_2021 <- fast_scraper_schedules(2021)
schedule_data_2020 <- fast_scraper_schedules(2020)
schedule_data_2019 <- fast_scraper_schedules(2019)

schedule_data <- rbind(schedule_data_2024,schedule_data_2023,schedule_data_2022,schedule_data_2021,
                       schedule_data_2020,schedule_data_2019)

#View(schedule_data)
```
Creation of Synthetic Data:
```{r}
test_sch <- schedule_data %>% select("game_id","season","week","gameday","away_team","home_team")

test_sch <- rename(test_sch, "game_date" = "gameday")

# Create the desired structure
expanded_df <- test_sch %>%
  # Create the first set of rows (home_team as posteam)
  mutate(posteam = home_team, defteam = away_team) %>%
  # Add the second set of rows (away_team as posteam)
  bind_rows(
    test_sch %>%
      mutate(posteam = away_team, defteam = home_team)
  )

pbp_data <- bind_rows(expanded_df, pbp_data)
```
Collecting Team Stats per Game:
```{r}
# Add a new column to classify explosive plays
pbp_data <- pbp_data %>%
  mutate(explosive_play = case_when(
    play_type == "run" & yards_gained >= 10 ~ "Explosive Run",
    play_type == "pass" & yards_gained >= 20 ~ "Explosive Pass",
    TRUE ~ "Non-Explosive"
  ))

# Summarize team statistics, focusing on points scored by the team
team_stats_for <- pbp_data %>%
  group_by(posteam, season, week, game_date) %>%
  summarize(
    points_for = max(posteam_score, na.rm = TRUE), # Points scored by the team
    yards_for = sum(yards_gained, na.rm = TRUE),
    plays_for = n(),
    avg_epa_for = mean(epa, na.rm = TRUE),
    success_rate_for = mean(success, na.rm = TRUE),
    pass_attempts_for = sum(play_type == "pass", na.rm = TRUE),
    run_attempts_for = sum(play_type == "run", na.rm = TRUE),
    pass_rate_for = pass_attempts_for / (pass_attempts_for + run_attempts_for),
    ex_runs_for = sum(explosive_play == "Explosive Run"),
    ex_pass_for = sum(explosive_play == "Explosive Pass"),
    ex_plays_for = sum(ex_runs_for + ex_pass_for),
    ex_pct_for = (ex_plays_for / plays_for))

team_stats_against <- pbp_data %>%
  group_by(defteam, season, week, game_date) %>%
  summarize(
    points_against = max(posteam_score, na.rm = TRUE), # Points scored by the team
    yards_against = sum(yards_gained, na.rm = TRUE),
    plays_against = n(),
    avg_epa_against = mean(epa, na.rm = TRUE),
    success_rate_against = mean(success, na.rm = TRUE),
    pass_attempts_against = sum(play_type == "pass", na.rm = TRUE),
    run_attempts_against = sum(play_type == "run", na.rm = TRUE),
    pass_rate_against = pass_attempts_against / (pass_attempts_against + run_attempts_against),
    ex_runs_against = sum(explosive_play == "Explosive Run"),
    ex_pass_against = sum(explosive_play == "Explosive Pass"),
    ex_plays_against = sum(ex_runs_against + ex_pass_against),
    ex_pct_against = (ex_plays_against / plays_against))

#View(team_stats_for)
#View(team_stats_against)

### Merging the Datasets

team_stats <- team_stats_for %>%
  left_join(team_stats_against, by = c("posteam" = "defteam","season" = "season","week" = "week",
                                       "game_date"="game_date"))

#View(team_stats)
```
Define Functions for Slide_DBL
```{r}
#Weighted EPA Against Function
weighted_epaf_function <- function(avg_epa_for, plays_for) {
  sum(avg_epa_for * plays_for) / sum(plays_for)
}

#Weighted EPA Against Function
weighted_epaa_function <- function(avg_epa_against, plays_against) {
  sum(avg_epa_against * plays_against) / sum(plays_against)
}

#Weighted Success Rate For Function
weighted_srf_function <- function(success_rate_for, plays_for) {
  sum(success_rate_for * plays_for) / sum(plays_for)
}

#Weighted Success Rate Against Function
weighted_sra_function <- function(success_rate_against, plays_against) {
  sum(success_rate_against * plays_against) / sum(plays_against)
}

#Weighted Pass Rate For Function
weighted_prf_function <- function(pass_rate_for, pass_attempts_for) {
  sum(pass_rate_for * pass_attempts_for) / sum(pass_attempts_for)
}

#Weighted Pass Rate Against Function
weighted_pra_function <- function(pass_rate_against, pass_attempts_against) {
  sum(pass_rate_against * pass_attempts_against) / sum(pass_attempts_against)
}

#Weighted Ex Play Rate For Function
weighted_xprf_function <- function(ex_pct_for, plays_for) {
  sum(ex_pct_for * plays_for) / sum(plays_for)
}

#Weighted Ex Play Rate Against Function
weighted_xpra_function <- function(ex_pct_against, plays_against) {
  sum(ex_pct_against * plays_against) / sum(plays_against)
}
```
