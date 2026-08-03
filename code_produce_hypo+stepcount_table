#R code to create hypometrics ID table
rm(list = ls())
#load packages for use

library(dplyr)

# Load Type 1 CGM/Fitbit data

fitbit <- readRDS("cgm_fitbit_metrics_type1.RDa")

# Sanity checks
dim(fitbit)
names(fitbit)
head(fitbit)

# Check required columns exist
required_fitbit_cols <- c(
  "Hypometrics_ID",
  "CGM_Timestamp",
  "Gl",
  "sum_steps_by_int"
)

missing_fitbit_cols <- setdiff(required_fitbit_cols, names(fitbit))

if (length(missing_fitbit_cols) > 0) {
  stop(
    paste(
      "Missing columns in fitbit data:",
      paste(missing_fitbit_cols, collapse = ", ")
    )
  )
}

# Checkpoint save
saveRDS(fitbit, "checkpoint_01_fitbit_type1_loaded.rds")

#sanitation, remove unused columns keep hypometrics id, sum of steps, timestamps and glucose readings. 

fitbit_final <- fitbit %>%
  select(
    Hypometrics_ID,
    CGM_Timestamp,
    Gl,
    sum_steps_by_int
  )

# Checkpoint save
saveRDS(fitbit_final, "checkpoint_02_fitbit_final.rds")

# Remove large object
rm(fitbit)

# separate day into 4 quarters: of morning,afternoon,evening,night.

fitbit_final <- fitbit_final %>%
  mutate(
    CGM_Timestamp = as.POSIXct(CGM_Timestamp),
    Date = as.Date(CGM_Timestamp),
    hour = as.numeric(format(CGM_Timestamp, "%H")),
    `Time window` = case_when(
      hour >= 6  & hour < 12 ~ "Morning",
      hour >= 12 & hour < 18 ~ "Afternoon",
      hour >= 18 & hour < 24 ~ "Evening",
      hour >= 0  & hour < 6  ~ "Night",
      TRUE ~ NA_character_
    )
  )

# Save checkpoint
saveRDS(fitbit_final, "checkpoint_03_with_time_windows.rds")

# Check time windows
table(fitbit_final$`Time window`, useNA = "ifany")

#sum of steps at moderate pace per minute (between 100- 139)

fitbit_final <- fitbit_final %>%
  mutate(
    moderate_minute = ifelse(
      sum_steps_by_int >= 100 & sum_steps_by_int < 140,
      1,
      0
    )
  )

# Save checkpoint
saveRDS(fitbit_final, "checkpoint_04_with_moderate_minutes.rds")

# Check moderate minute count
table(fitbit_final$moderate_minute, useNA = "ifany")

#create a glucose table to compare eveything 
# One row per participant, date and time window.

final_output <- fitbit_final %>%
  group_by(
    Hypometrics_ID,
    Date,
    `Time window`
  ) %>%
  summarise(
    `Average glucose` = mean(Gl, na.rm = TRUE),
    `Total steps` = sum(sum_steps_by_int, na.rm = TRUE),
    `Total minutes of moderate steps` = sum(moderate_minute, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  mutate(
    `Time window` = factor(
      `Time window`,
      levels = c("Morning", "Afternoon", "Evening", "Night"),
      ordered = TRUE
    )
  ) %>%
  arrange(
    Hypometrics_ID,
    Date,
    `Time window`
  )

# Save checkpoint
saveRDS(final_output, "checkpoint_05_final_output.rds")

# Remove large minute-level table now that summary is created
rm(fitbit_final)

# Check table
head(final_output)
dim(final_output)

# Save steps/glucose output
write.csv(
  final_output,
  "Type_1_final_table.csv",
  row.names = FALSE
)


#  Load night hypo files


hypos_3_9 <- read.csv("hypos_3.9_night.csv", stringsAsFactors = FALSE)
hypos_3_0 <- read.csv("hypos_3.0_night.csv", stringsAsFactors = FALSE)

# Check everything is there 
dim(hypos_3_9)
names(hypos_3_9)
head(hypos_3_9)

dim(hypos_3_0)
names(hypos_3_0)
head(hypos_3_0)

# Check required hypo columns exist
required_hypo_cols <- c(
  "id",
  "sdh_interval",
  "sdh_duration_mins",
  "sdh_nadir"
)

missing_hypo_3_9_cols <- setdiff(required_hypo_cols, names(hypos_3_9))
missing_hypo_3_0_cols <- setdiff(required_hypo_cols, names(hypos_3_0))

if (length(missing_hypo_3_9_cols) > 0) {
  stop(
    paste(
      "Missing columns in hypos_3_9:",
      paste(missing_hypo_3_9_cols, collapse = ", ")
    )
  )
}

if (length(missing_hypo_3_0_cols) > 0) {
  stop(
    paste(
      "Missing columns in hypos_3_0:",
      paste(missing_hypo_3_0_cols, collapse = ", ")
    )
  )
}

# -----------------------------
# 8. Clean level 1 night hypo file
# -----------------------------
# Level 1 hypo = 3.9 mmol/L

hypos_3_9_clean <- hypos_3_9 %>%
  mutate(
    Date = as.Date(substr(sdh_interval, 1, 10))
  ) %>%
  rename(
    Hypometrics_ID = id
  ) %>%
  group_by(
    Hypometrics_ID,
    Date
  ) %>%
  summarise(
    hypo_lvl1_night = 1,
    number_lvl1_hypos = n(),
    total_lvl1_hypo_duration_mins = sum(sdh_duration_mins, na.rm = TRUE),
    lowest_lvl1_nadir = min(sdh_nadir, na.rm = TRUE),
    .groups = "drop"
  )

saveRDS(hypos_3_9_clean, "checkpoint_06_hypos_3_9_clean.rds")

head(hypos_3_9_clean)

#clean the hypometric data for nocturnal glucose

hypos_3_0_clean <- hypos_3_0 %>%
  mutate(
    Date = as.Date(substr(sdh_interval, 1, 10))
  ) %>%
  rename(
    Hypometrics_ID = id
  ) %>%
  group_by(
    Hypometrics_ID,
    Date
  ) %>%
  summarise(
    hypo_lvl2_night = 1,
    number_lvl2_hypos = n(),
    total_lvl2_hypo_duration_mins = sum(sdh_duration_mins, na.rm = TRUE),
    lowest_lvl2_nadir = min(sdh_nadir, na.rm = TRUE),
    .groups = "drop"
  )

saveRDS(hypos_3_0_clean, "checkpoint_07_hypos_3_0_clean.rds")

head(hypos_3_0_clean)


# Join night hypo data to steps/glucose table
# This matches by Hypometrics_ID and Date.
# The night hypo flag appears for all time windows on that date.

final_output_with_hypos <- final_output %>%
  left_join(
    hypos_3_9_clean,
    by = c("Hypometrics_ID", "Date")
  ) %>%
  left_join(
    hypos_3_0_clean,
    by = c("Hypometrics_ID", "Date")
  ) %>%
  mutate(
    hypo_lvl1_night = ifelse(is.na(hypo_lvl1_night), 0, hypo_lvl1_night),
    hypo_lvl2_night = ifelse(is.na(hypo_lvl2_night), 0, hypo_lvl2_night),
    number_lvl1_hypos = ifelse(is.na(number_lvl1_hypos), 0, number_lvl1_hypos),
    number_lvl2_hypos = ifelse(is.na(number_lvl2_hypos), 0, number_lvl2_hypos),
    total_lvl1_hypo_duration_mins = ifelse(is.na(total_lvl1_hypo_duration_mins), 0, total_lvl1_hypo_duration_mins),
    total_lvl2_hypo_duration_mins = ifelse(is.na(total_lvl2_hypo_duration_mins), 0, total_lvl2_hypo_duration_mins)
  ) %>%
  mutate(
    `Time window` = factor(
      `Time window`,
      levels = c("Morning", "Afternoon", "Evening", "Night"),
      ordered = TRUE
    )
  ) %>%
  arrange(
    Hypometrics_ID,
    Date,
    `Time window`
  )

# Save checkpoint
saveRDS(final_output_with_hypos, "checkpoint_08_final_output_with_hypos.rds")

# Check final joined table
head(final_output_with_hypos)
dim(final_output_with_hypos)

# Save final joined output
write.csv(
  final_output_with_hypos,
  "Type_1_final_table_with_night_hypos.csv",
  row.names = FALSE
)
