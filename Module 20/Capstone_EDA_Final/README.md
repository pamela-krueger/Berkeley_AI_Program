### Income, Food Planning, and Food-at-Home (FAH) Acquisition in US Households


#### Executive summary
This project uses the USDA National Household Food Acquisition and Purchase Survey (FoodAPS) to ask whether household income, grocery list use and other food planning behaviors are associated with lower FAH acquisition. 

After cleaning and merging three FoodAPS files into a 4,826 household dataset, exploratory analysis and a baseline linear regression show that these characteristics are associated with acquisition, but in the opposite direction from the hypothesis: households with higher incomes, households that shop with a grocery list more often, and households that prepare more dinners at home all acquire more food at home per person, not less. Eating out more, larger household size and food insecurity are associated with lower acquisition. 

These relationships are real but modest: the baseline model explains about 6% of the variation in weekly spending per person, and the best preliminary model explains about 7%.

#### Rationale
According to the Project Drawdown website, “More than one-third of all food produced for human consumption is lost or wasted before it can be eaten.” This is a staggering amount of waste. Moreover, the greenhouse gas emissions related to the production and distribution of this wasted food are high and also wasted. These societal losses have enormous impacts at the household and country levels, as well as on the planet as a whole given climate change impacts. Understanding more about which characteristics of households (income and behaviors) are more strongly associated with higher acquisition — and thus a higher potential for food waste (in the US but potentially with some generalizability to other similar countries) — could help policymakers implement measures to curb waste. For example, perhaps policymakers could use this information to target information campaigns towards households more likely to waste with tips on how to reduce food waste at home.

#### Research Question
Are income, grocery list use and other food planning behaviors associated with lower FAH acquisition among US households?

#### Data Sources
USDA National Household Food Acquisition and Purchase Survey (FoodAPS) public use files — https://www.ers.usda.gov/data-products/foodaps-national-household-food-acquisition-and-purchase-survey (CSV files. documentation is also found at this link).

> Information from the FoodAPS User Guide: "The National Household Food Acquisition and Purchase Survey (FoodAPS) collected information on foods purchased or otherwise acquired, and the prices and nutrient characteristics of those foods, for a nationally representative sample of U.S. households. Data on factors expected to affect food acquisition decisions, such as household size and composition, demographic characteristics, income, participation in Federal food assistance programs, and dietary restrictions, were also collected."

Files used (in `Data/CSV_data_files/`):
- `faps_household_puf.csv` — one row per household
- `faps_fahevent_puf.csv` — one row per FAH acquisition event
- `faps_access_puf.csv` — one row per household, describing the local food environment

Key variables: 
-household spending (`TOTALPAID`) and acquisition events (`EVENTID`) normalized by household size (`HHSIZE`
-income as a percentage of the poverty line (`PCTPOVGUIDEHH_R`)
-grocery-list use (`GROCERYLISTFREQ`) 
-home meal routines (`NDINNERSOUTHH`, `NMEALSHOME`, `NMEALSTOGETHER`) 
-food security (`ADLTFSCAT`, `FOODSUFFICIENT`)
-food assistance (`SNAPNOWHH`, `WICHH`)
-transportation (`ANYVEHICLE`, `CARACCESS`)
-primary store distance and type
-local food access
-region
-rurality
-survey month

#### Methodology
See Jupyter Notebook called "Capstone_EDA_final" for code and analysis.

1. Data cleaning: FoodAPS missing value codes were recoded to NaN. Valid skips were recoded logically (ex: households with no WIC eligible member set to 0 for WIC)
"Meals together" set to 0 for one person households Missing event spending (1% of events) was imputed with the mean for the same store type
Missing store distance with the mean distance
No duplicate rows or duplicate keys were found.

2. Feature engineering: Event level data was aggregated to the household level (total spending, number of events, acquisition days, share of free/SNAP/WIC events, number of store types) and merged one to one with the household and access files. 
New features include spending and events per person, a food-insecurity flag, income as a multiple of the poverty line, season, region and vehicle-access categories, and a log1p transform of spending per person as the model outcome (since there were many zero spending events).

3. Outlier analysis: Boxplots and the 1.5×IQR rule identified a long right tail of high spenders. These were judged to be genuine large shopping trips and kept, with the log transform limiting their influence. About 10% of households recorded $0 of FAH spending during their survey week.

4. EDA: distributions of outcomes and categorical variables, spending by grocery list use, income group, food security and home cooking, income and household size relationships, and a correlation matrix.

5. Baseline model: a multiple linear regression predicting log weekly FAH spending per person, compared against a naive model that always predicts the mean. **

6. Evaluation metrics: test-set mean squared error (MSE), with RMSE and R² also reported. 
MSE is the loss the model minimizes and penalizes large errors
R² shows how much of the household-to-household variation the predictors explain, which is the heart of the research question.



#### Results
EDA:
- Median weekly FAH spending per person rises steadily with grocery-list use and with income.
- Households that prepare more dinners at home spend more
- households with very low food security spend the least
- Spending per person falls with household size

Baseline model - linear regression:

| Model | Test MSE | Test RMSE | Test R² |
|---|---|---|---|
| Naive mean (reference) | 1.969 | 1.403 | 0.000 |
| Linear Regression (baseline) | 1.859 | 1.364 | 0.055 |


- Evaluation metrics: the baseline linear regression has a test MSE of 1.859 versus 1.969 for the naive mean model, which is about a 6% reduction in error. Its test R² is 0.055, meaning income, food-planning behaviors and the controls together explain about 6% of the household-to-household variation in weekly FAH spending per person. Train and test R² are close, so the model is not overfitting.
- An RMSE of 1.4 on the log scale is large and it means typical predictions are off by a multiplicative factor of several. The predicted-vs-actual plot shows thta the model predicts a narrow band and cannot reach the households with zero spending  which produce the separate cluster of residuals.
- More dinners prepared at home, higher income and more frequent grocery list use are each associated with higher FAH spending per person. Larger household size, food insecurity and eating out more are associated with lower spending. Households surveyed in winter spent less than those surveyed in fall, and households in the West spent more than those in the Midwest.

**Research question:** Are income, grocery-list use and other food planning behaviors associated with lower food-at-home (FAH) acquisition among US households?

**Preliminary answer:** they are associated with acquisition, but in the opposite direction from the hypothesis, and only modestly.

1. Higher income, more grocery-list use, and more meals prepared at home all go with more FAH acquisition, not less
2. Eating out more is associated with lower FAH acquisition
3. Household size, food insecurity, season and region matter at least as much as the income and planning variables. Larger households spend less per person (economies of scale), food-insecure households spend less, households surveyed in winter spent less, and Western households spent more.
4. Interpretation for the food-waste motivation: because FoodAPS measures acquisition rather than waste, higher spending among planners and home cooks does not by itself mean more waste. These households may simply eat more of their meals from food bought for home. A single week of acquisition data also captures a lot of noise where about 10% of households bought nothing that week, which limits how much any household characteristic can explain.

#### Ideas for Next Steps
- Model the zero spending weeks separately since about 10% of households had zero of FAH spending. A two part (hurdle) model with a classifier for whether a household acquired food that week plus a regression for how much among those that did spend should fit far better than one linear model.
- Try Ridge and Lasso alpha grids, and try tree based models (random forest, gradient boosting) that capture interactions such as income × grocery list use.
- Use cross validation for final model selection rather than a single train/test split


