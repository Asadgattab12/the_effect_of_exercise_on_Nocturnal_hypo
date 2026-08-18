# Clear the environment before starting, just to make sure nothing is left
rm(list = ls())
#load packages for use
library(tibble)
library(forestplot)

# set tibble data frame
# Each row here corresponds to one model's output pulled from the regression tables
base_data <- tibble(
  #odds ratios for each of the four models
  mean = c(
    1.0196466,
    1.0236896,
    1.0019906,
    1.0041439
  ),
  # Lower bound of the 95% Wald CI for each OR
  lower = c(
    1.0008025,
    1.0059870,
    0.9979388,   
    1.0009090
  ),
  # Upper bound of the 95% Wald CI for each OR
  upper = c(
    1.0388454,
    1.0417038,
    1.0060589,
    1.0073893
  ),
  # numbering each model
  model = c("1", "2", "3", "4"),
  # labels to identify what is shown
  predictor = c(
    "Morning total steps",
    "Afternoon total steps",
    "Morning moderate stepping",
    "Afternoon moderate stepping"
  ),
  # Total steps models are scaled per 1,000 steps, moderate stepping is per minute
  unit = c(
    "Per 1,000 steps",
    "Per 1,000 steps",
    "Per minute",
    "Per minute"
  ),
  # Pre-formatted Odd ratios strings for the label column, rounded to 3 dp
  OR_CI = c(
    "1.020 (1.001–1.039)",
    "1.024 (1.006–1.042)",
    "1.002 (0.998–1.006)",
    "1.004 (1.001–1.007)"
  ),
  # p-values pulled from the same models, kept as string
  p_value = c("0.041", "0.009", "0.336", "0.012")
)

# Build the forest plot itself
forest_plot <- base_data |>
  forestplot(
    # These are the columns that get printed as text alongside the plot
    labeltext = c(model, predictor, unit, OR_CI, p_value),
    graph.pos = "right",     # puts the forest plot on the right, labels on the left
    zero = 1,                # reset null value as 1 instead of 0
    clip = c(0.995, 1.045),  # trims the CI whiskers to this range so large outliers don't ruin plot distribution
    xticks = c(0.995, 1.05, 1.015, 1.025, 1.035, 1.045),  # manual tick marks for x-axis
    xlog = TRUE,              # plot on a log scale.
    title = "Associations between morning and afternoon activity and the effect on nocturnal hypoglycaemia",
    xlab = "Odds ratio"
  ) |>
  # Set the visual style - blue boxes/lines throughout, nothing fancy
  fp_set_style(
    box = "royalblue",
    line = "darkblue",
    summary = "royalblue"
  ) |>
  # Add proper column headers so the label columns don't just show
  fp_add_header(
    model = "Model",
    predictor = "Activity predictor",
    unit = "Unit of increase",
    OR_CI = "OR (95% CI)",
    p_value = "p-value"
  ) |>
  # Alternate row shading to make it easier to track each row across
  # to the plotted forest on the right
  fp_set_zebra_style("#EFEFEF")

# display the forest plot
print(forest_plot)

