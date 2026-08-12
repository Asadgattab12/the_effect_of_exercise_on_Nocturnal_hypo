#Reset workspace and load packages
# rm (list = ls())
# library(dplyr)
# library(lme4)

# Load data
final_output_with_hypos <- read.csv(
  "Type_1_final_table_with_night_hypos.csv",
  stringsAsFactors = FALSE
)

# Create nocturnal hypoglycaemia outcome for Level 1 nocturnal hypoglycaemia
final_output_with_hypos <- final_output_with_hypos %>%
  mutate(
    nocturnal_hypo = ifelse(hypo_lvl1_night == 1, 1, 0)
  )

# Create one row per participant/date
daily_moderate <- final_output_with_hypos %>%
  select(
    Hypometrics_ID,
    Date,
    Time.window,
    Total.minutes.of.moderate.steps,
    nocturnal_hypo
  ) %>%
  # Reshape from long to wide format so each participant-date has one row 
  ##and each time window has a separate moderate-activity minutes column
  tidyr::pivot_wider(
    id_cols = c(Hypometrics_ID, Date, nocturnal_hypo),
    names_from = Time.window,
    values_from = Total.minutes.of.moderate.steps,
    names_prefix = "moderate_minutes_"
  ) %>%
  #Replace missing Morning and Afternoon moderate-minute values with zero 
  ##and convert participant ID to a factor for mixed-effects modelling
  mutate(
    moderate_minutes_Morning = ifelse(
      is.na(moderate_minutes_Morning), 0, moderate_minutes_Morning
    ),
    moderate_minutes_Afternoon = ifelse(
      is.na(moderate_minutes_Afternoon), 0, moderate_minutes_Afternoon
    ),
    Hypometrics_ID = as.factor(Hypometrics_ID)
  )

cat("Participant-days:", nrow(daily_moderate), "\n")
cat(
  "Participants:",
  length(unique(daily_moderate$Hypometrics_ID)),
  "\n\n"
)

# Model 3: morning moderate-activity minutes with participant random intercept
model3 <- glmer(
  nocturnal_hypo ~ moderate_minutes_Morning +
    (1 | Hypometrics_ID),
  data = daily_moderate,
  family = binomial(link = "logit"),
  control = glmerControl(
    optimizer = "bobyqa",
    optCtrl = list(maxfun = 2e5)
  )
)
cat("Model 3: Morning moderate minutes")
print(summary(model3))

# Calculate odds ratios and 95% Wald confidence intervals
model3_CI <- confint(
  model3,
  parm = "beta_",
  method = "Wald"
)
model3_OR <- exp(cbind(
  OR = fixef(model3),
  model3_CI
))
cat("\nOdds ratios and 95% Wald CI (Model 3):\n")
print(model3_OR)

#Model 4: afternoon moderate-activity minutes with participant random intercept.
model4 <- glmer(
  nocturnal_hypo ~ moderate_minutes_Afternoon +
    (1 | Hypometrics_ID),
  data = daily_moderate,
  family = binomial(link = "logit"),
  control = glmerControl(
    optimizer = "bobyqa",
    optCtrl = list(maxfun = 2e5)
  )
)
cat("Model 4: Afternoon moderate minutes")
print(summary(model4))

model4_CI <- confint(
  model4,
  parm = "beta_",
  method = "Wald"
)
model4_OR <- exp(cbind(
  OR = fixef(model4),
  model4_CI
))
cat("\nOdds ratios and 95% Wald CI (Model 4):\n")
print(model4_OR)


# Save moderate stepping models for morning and afternoon 

saveRDS(model3, "model3_morning_moderate_minutes.rds")
saveRDS(model4, "model4_afternoon_moderate_minutes.rds")

# Export odds ratios and 95% confidence intervals for use in the report
write.csv(
  model3_OR,
  "model3_morning_moderate_minutes_wald_OR.csv"
)
write.csv(
  model4_OR,
  "model4_afternoon_moderate_minutes_wald_OR.csv"
)

write.csv(
  daily_moderate,
  "daily_moderate_minutes_dataset.csv",
  row.names = FALSE
)

