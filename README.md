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
