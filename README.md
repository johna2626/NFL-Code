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
