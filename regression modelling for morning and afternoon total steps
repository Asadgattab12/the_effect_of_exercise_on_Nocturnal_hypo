# Mixed-effects logistic regression analysis
# Association of morning and afternoon total step counts against
# Level 1 nocturnal hypoglycaemia in participants with type 1 diabetes


# Remove existing objects to begin with a clean workspace
rm(list = ls())

# Load packages required
library(dplyr)
library(lme4)


# Load the integrated participant-date dataset containing Fitbit activity
# summaries and nocturnal hypoglycaemia outcomes
final_output_with_hypos <- read.csv(
  "Type_1_final_table_with_night_hypos.csv",
  stringsAsFactors = FALSE
)

# Create a binary outcome variable:
# 1 = at least one Level 1 nocturnal hypo
# 0 = no Level 1 nocturnal hypo
final_output_with_hypos <- final_output_with_hypos %>%
  mutate(
    nocturnal_hypo = ifelse(hypo_lvl1_night == 1, 1, 0)
  )


# Reshape the data to participant-day level


# Convert the time-window data from long to wide format so that each row
# represents one participant-day and morning and afternoon step counts are
# stored in separate predictor columns
daily <- final_output_with_hypos %>%
  select(
    Hypometrics_ID,
    Date,
    Time.window,
    Total.steps,
    nocturnal_hypo
  ) %>%
  tidyr::pivot_wider(
    id_cols = c(Hypometrics_ID, Date, nocturnal_hypo),
    names_from = Time.window,
    values_from = Total.steps,
    names_prefix = "steps_"
  ) %>%
  mutate(
    # Replace absent step totals with zero
    steps_Morning = ifelse(
      is.na(steps_Morning), 0, steps_Morning
    ),
    steps_Afternoon = ifelse(
      is.na(steps_Afternoon), 0, steps_Afternoon
    ),
# Treat participant ID as a categorical grouping variable for
# participant random intercept
    Hypometrics_ID = as.factor(Hypometrics_ID)
  )

#summarising the final models:

# Report the number of participant entries, unique participants and the
# percentage of participant entries with 1 or 0
cat("Participant-days:", nrow(daily), "\n")

cat(
  "Participants:",
  length(unique(daily$Hypometrics_ID)),
  "\n"
)

cat(
  "Nocturnal hypoglycaemia rate:",
  round(mean(daily$nocturnal_hypo) * 100, 2),
  "%\n\n"
)

#rescale step count predictors

# Express step counts in units of 1,000 steps so that the resulting odds ratios
# represent the change in odds associated with each additional 1,000 steps
daily <- daily %>%
  mutate(
    morning_steps_k = steps_Morning / 1000,
    afternoon_steps_k = steps_Afternoon / 1000
  )


# 5. Model 1: morning total steps


# Fit a mixed-effects logistic regression model:
# - Level 1 nocturnal hypoglycaemia as the binary outcome
# - Morning steps per 1,000 as the fixed-effect predictor
model1 <- glmer(
  nocturnal_hypo ~ morning_steps_k +
    (1 | Hypometrics_ID),
  data = daily,
  family = binomial(link = "logit"),
  control = glmerControl(
    # Use the bobyqa optimiser and increase the maximum number of function
    # evaluations to support model convergence
    optimizer = "bobyqa",
    optCtrl = list(maxfun = 2e5)
  )
)

# print full model 1
cat("\nMODEL 1: Morning total steps\n")
print(summary(model1))

# Calculate 95% Wald confidence intervals for the fixed-effect coefficients
# and exponentiate the log-odds estimates and intervals to obtain odds ratios
model1_CI <- confint(
  model1,
  parm = "beta_",
  method = "Wald"
)

model1_OR <- exp(cbind(
  OR = fixef(model1),
  model1_CI
))

# Display odds ratios and their 95% Wald confidence intervals
cat("\nOdds ratios and 95% Wald confidence intervals (Model 1):\n")
print(model1_OR)


# 6. Model 2: afternoon total steps


# Fit a second mixed-effects logistic regression model with
# - Level 1 nocturnal hypoglycaemia as the binary outcome
# - Afternoon steps per 1,000 as the fixed-effect predictor

model2 <- glmer(
  nocturnal_hypo ~ afternoon_steps_k +
    (1 | Hypometrics_ID),
  data = daily,
  family = binomial(link = "logit"),
  control = glmerControl(
    # Apply the same optimiser settings used for Model 1
    optimizer = "bobyqa",
    optCtrl = list(maxfun = 2e5)
  )
)

# Display the complete afternoon-step model summary
cat("\nMODEL 2: Afternoon total steps\n")
print(summary(model2))

# Calculate 95% Wald confidence intervals for the fixed-effect coefficients
# to obtain odds ratios
model2_CI <- confint(
  model2,
  parm = "beta_",
  method = "Wald"
)

model2_OR <- exp(cbind(
  OR = fixef(model2),
  model2_CI
))

# Display odds ratios and their 95% Wald confidence intervals
cat("\nOdds ratios and 95% Wald confidence intervals (Model 2):\n")
print(model2_OR)

# Save the fitted model objects so that they can be reloaded without refitting
saveRDS(
  model1,
  "model1_morning_steps.rds"
)

saveRDS(
  model2,
  "model2_afternoon_steps.rds"
)

write.csv(
  model1_OR,
  "model1_morning_steps_wald_OR.csv"
)

write.csv(
  model2_OR,
  "model2_afternoon_steps_wald_OR.csv"
)

# Export the participant-day used to fit Models 1 and 2
write.csv(
  daily,
  "daily_model_dataset.csv",
  row.names = FALSE
)
