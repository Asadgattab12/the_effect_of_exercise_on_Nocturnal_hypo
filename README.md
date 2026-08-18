##The effect of morning and afternoon exercise on nocturnal hypoglycaemia on type 1 diabetes patients using the hypo metrics data set##

##my final independent project for my MSC in Bioinformatics

Overview of the Type 1 Diabetes Exercise-Timing Analysis Pipeline

This repository implements a data-analysis pipeline investigating the relationship between the timing of physical activity and nocturnal hypoglycaemia in adults with type 1 diabetes (T1D). Fitbit step-count data and continuous glucose monitoring (CGM) data were processed to compare morning and afternoon activity and determine whether either period was associated with an increased likelihood of Level 1 nocturnal hypoglycaemia.

The workflow consists of five main stages:

Pre-processing the CGM and Fitbit data.
Classifying physical activity by time window and stepping intensity.
Integrating nocturnal hypoglycaemia events with the activity data.
Performing mixed-effects logistic regression analysis.
Producing participant summaries, boxplots and a forest plot.

The final analysis included 229 participants with T1D and 15,858 participant-day observations. The repeated daily observations contributed by each participant were accounted for using mixed-effects logistic regression models with participant identifier included as a random intercept.

CGM and Fitbit Data Preparation – 01_prepare_activity_and_hypoglycaemia_data.R

This stage prepares the minute-level CGM and Fitbit data for analysis. The input file, cgm_fitbit_metrics_type1.RDa, contains measurements for participants with T1D. The script first checks that the required variables are present, including participant identifier, CGM timestamp, glucose concentration and steps recorded during each interval.

Only the variables required for the analysis are retained. The CGM timestamp is converted into a date and hour, allowing every record to be assigned to one of four six-hour time windows:

Morning: 06:00–11:59.

Afternoon: 12:00–17:59.

Evening: 18:00–23:59.

Night: 00:00–05:59.

The analysis focuses primarily on the morning and afternoon periods. These windows were selected to allow activity performed earlier and later in the day to be compared consistently.

Moderate stepping is identified using a cadence threshold of 100–139 steps per minute. Each minute within this range is coded as one moderate-stepping minute. The data are then grouped by participant, date and time window to calculate average glucose, total steps and total moderate-stepping minutes.

The summarised activity dataset is exported as Type_1_final_table.csv. Checkpoint files are also created throughout the script, allowing the analysis to be resumed without repeatedly loading and processing the full minute-level dataset. This was particularly useful because the original CGM and Fitbit dataset was large and required substantial memory.

Nocturnal Hypoglycaemia Integration:

The next stage integrates nocturnal hypoglycaemia events with the summarised activity data. Two event files are used:

hypos_3.9_night.csv for events below 3.9 mmol/L.
hypos_3.0_night.csv for events below 3.0 mmol/L.

The script checks that each file contains participant identifier, event interval, duration and glucose nadir. Event dates are extracted from the interval variable, and participant identifiers are renamed so that they are consistent with the Fitbit dataset.

Events are summarised by participant and date. For each threshold, the pipeline records whether an event occurred, the number of events, their total duration and the lowest glucose nadir. The nocturnal-event summaries are then joined to the activity dataset using Hypometrics_ID and Date.

Participant-days with no recorded hypoglycaemic event are assigned a value of zero rather than being left as missing. This produces the final integrated dataset, Type_1_final_table_with_night_hypos.csv, which is used in all subsequent modelling and visualisation scripts.

The primary outcome used in the statistical models is Level 1 nocturnal hypoglycaemia:
1: at least one nocturnal hypoglycaemic event occurred.
0: no nocturnal hypoglycaemic event occurred.

The supplied nocturnal hypoglycaemia files were treated as already assigned to the participant date required for comparison with the corresponding daytime activity.

Total-Steps Mixed-Effects Models – 02_total_steps_models.R

This script evaluates the associations between morning and afternoon total steps and Level 1 nocturnal hypoglycaemia.

The final integrated dataset is reshaped from long to wide format so that each row represents one participant-day. Morning and afternoon step totals are placed into separate predictor columns. Missing morning or afternoon step totals are replaced with zero, and participant identifier is converted to a factor for mixed-effects modelling.

Total steps are divided by 1,000 before modelling. This makes the resulting odds ratios clinically easier to interpret because they represent the change in the odds of nocturnal hypoglycaemia for each additional 1,000 steps.

Two models are fitted using glmer() from the lme4 package:

# Model 1: morning total steps
nocturnal_hypo ~ morning_steps_k + (1 | Hypometrics_ID)
# Model 2: afternoon total steps
nocturnal_hypo ~ afternoon_steps_k + (1 | Hypometrics_ID)

Both models use a binomial distribution with a logit link. Participant identifier is included as a random intercept to account for repeated observations from the same individual. The bobyqa optimiser is used with a maximum of 200,000 function evaluations to support model convergence.

The fitted model objects are saved as .rds files. Odds ratios and 95% Wald confidence intervals are exported as CSV files, alongside the participant-day dataset used in the models.

The outputs include:

model1_morning_steps.rds.
model2_afternoon_steps.rds.

model1_morning_steps_wald_OR.csv.
model2_afternoon_steps_wald_OR.csv.

daily_model_dataset.csv.

Moderate-Stepping Mixed-Effects Models – 03_moderate_stepping_models.R

This script examines whether the number of moderate-stepping minutes recorded during the morning or afternoon is associated with Level 1 nocturnal hypoglycaemia.

The data are again reshaped to one row per participant-day, with separate variables for morning and afternoon moderate-stepping minutes. Missing activity values are replaced with zero, and participant identifier is included as the random-effect grouping variable.

Two more models are fitted:

# Model 3: morning moderate-stepping minutes
nocturnal_hypo ~ moderate_minutes_Morning + (1 | Hypometrics_ID)
# Model 4: afternoon moderate-stepping minutes
nocturnal_hypo ~ moderate_minutes_Afternoon + (1 | Hypometrics_ID)

Unlike the total-step models, the odds ratios from these models represent the change in odds associated with each additional minute of moderate stepping. The models use the same binomial-logit structure, random intercept, optimiser and Wald confidence-interval method as Models 1 and 2.

The outputs include:

model3_morning_moderate_minutes.rds.
model4_afternoon_moderate_minutes.rds.

model3_morning_moderate_minutes_wald_OR.csv.
model4_afternoon_moderate_minutes_wald_OR.csv.

daily_moderate_minutes_dataset.csv.

Activity Boxplots – 04_activity_boxplots.R

This stage produces descriptive statistics and boxplots comparing activity distributions according to nocturnal hypoglycaemia status. The final dataset is restricted to the morning and afternoon periods, and the binary outcome is converted into the labels No nocturnal hypo and Nocturnal hypo.

The first quartile, median and third quartile are calculated for total steps and moderate-stepping minutes within each time window and hypoglycaemia group. These summaries provide the values used to describe the activity distributions in the Results section.

Two faceted boxplots are produced using ggplot2. The first compares total steps, while the second compares moderate-stepping minutes. Morning and afternoon results are displayed in separate panels, allowing the timing and hypoglycaemia groups to be compared visually.

The plots are exported at 300 dpi as:

figure_boxplot_steps.png.
figure_boxplot_moderate_minutes.png.

The plots contain substantial overlap between the hypoglycaemia and non-hypoglycaemia groups, reflecting the small effect sizes observed in the regression models. High-value outliers are retained because they are plausible in free-living step-count data, although their visual prominence is reduced using partial transparency.

Forest Plot – 05_forest_plot.R

This script combines the results of all four mixed-effects models into a single forest plot. The forestplot package is used to display the model number, activity predictor, unit of increase, odds ratio, 95% confidence interval and p-value.

A reference line is positioned at an odds ratio of 1.00, representing no association. Confidence intervals crossing this value indicate that the association is not statistically significant at the 0.05 threshold. Alternating row shading is applied to make it easier to track each model across the figure.

The values in the forest-plot data frame are entered from the final regression tables. If the models are rerun or changed, these values should be checked and updated to prevent the figure from becoming inconsistent with the reported model outputs.

Participant Characteristics – 06_participant_characteristics.py

This Python script creates the participant-characteristics table using the baseline participant dataset and the final analysis dataset. Participant identifiers are matched to retain only individuals represented in the analysis, and duplicate participant records are removed.

Age and HbA1c are summarised using means and standard deviations. Sex and ethnicity are reported using frequencies and percentages. HbA1c values originally recorded as percentages are converted to mmol/mol before the summary table is produced. The final table describes the 229 participants included in the mixed-effects models.

Main Findings

Both total-step models showed statistically significant but small positive associations with Level 1 nocturnal hypoglycaemia. Each additional 1,000 morning steps was associated with approximately 2.0% higher odds, while each additional 1,000 afternoon steps was associated with approximately 2.4% higher odds.

Morning moderate-stepping minutes were not significantly associated with nocturnal hypoglycaemia. Afternoon moderate stepping showed a small positive association, with each additional minute associated with approximately 0.4% higher odds.

The results represent associations rather than evidence that physical activity directly caused nocturnal hypoglycaemia. Insulin dosing, carbohydrate intake, exercise duration, baseline glucose and individual insulin sensitivity were not included in the models and may also influence the outcome.

Required Software and Packages:

The R analysis uses the following packages:

dplyr.
tidyr.
lme4.
ggplot2.
scales.
tibble.
forestplot.

The participant-characteristics analysis uses Python with:

pyreadr.
pandas.
numpy.
tableone.

Recommended Running Order

The scripts should be run in the following order:

01_prepare_activity_and_hypoglycaemia_data.R.

02_total_steps_models.R.

03_moderate_stepping_models.R.

04_activity_boxplots.R.

05_forest_plot.R.

06_participant_characteristics.py.

Each R script begins by clearing the environment with rm(list = ls()). They are therefore intended to run independently, with the required input files stored in the working directory or referenced using the correct file paths.

##Notes and Troubleshooting:

This pipeline was developed as part of an academic project examining real-time physical activity and nocturnal glycaemia in T1D. The original minute-level dataset was large, which caused memory and processing difficulties during development. Intermediate checkpoint files were therefore created so that completed processing stages could be reloaded without restarting the entire workflow.

Column names containing spaces were converted by read.csv() into names containing full stops. For example, Time window became Time.window, Total steps became Total.steps, and Total minutes of moderate steps became Total.minutes.of.moderate.steps. The modelling and plotting scripts use these converted names.

The forest plot uses manually entered estimates. Any changes to the regression models must also be applied to the forest-plot values. Model convergence messages, dataset dimensions and participant counts should be checked whenever the pipeline is rerun.

Author

Asad A. K. Gattab
MSc Bioinformatics
University of Leicester
