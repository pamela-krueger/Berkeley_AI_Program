# Income, Food Planning, and Food-at-Home (FAH) Acquisition in US Households

## Executive summary
This project uses the USDA National Household Food Acquisition and Purchase Survey (FoodAPS) to ask whether household income, grocery list use and other food planning behaviors are associated with lower FAH acquisition. 

After cleaning and merging three FoodAPS files into a 4,826 household dataset, exploratory analysis and eight models were used: linear, polynomial, Ridge and Lasso regression, a decision tree, a random forest, gradient boosting and a neural network. The models show that these characteristics are associated with acquisition, but in the opposite direction from the hypothesis. That is, households with higher incomes, households that shop with a grocery list more often, and households that prepare more dinners at home all acquire more food at home per person, not less. Eating out more, larger household size and food insecurity are associated with lower acquisition. 

These relationships are real but modest. The baseline linear regression explains about 6% of the variation in weekly spending per person, and the best model, which was gradient boosting, explains about 8%. More complex models improve the fit only a little, so the main limitation is the data rather than the choice of model.

## Rationale
According to the Project Drawdown website, “More than one-third of all food produced for human consumption is lost or wasted before it can be eaten.” This is a staggering amount of waste. Moreover, the greenhouse gas emissions related to the production and distribution of this wasted food are high and also wasted. These societal losses have enormous impacts at the household and country levels, as well as on the planet as a whole given climate change impacts. Understanding more about which characteristics of households (income and behaviors) are more strongly associated with higher acquisition — and thus a higher potential for food waste (in the US but potentially with some generalizability to other similar countries) — could help policymakers implement measures to curb waste. For example, perhaps policymakers could use this information to target information campaigns towards households more likely to waste with tips on how to reduce food waste at home.

## Research Question
Are income, grocery list use and other food planning behaviors associated with lower FAH acquisition among US households?

## Data Sources
USDA National Household Food Acquisition and Purchase Survey (FoodAPS) public use files — https://www.ers.usda.gov/data-products/foodaps-national-household-food-acquisition-and-purchase-survey (CSV files. Documentation is also found at this link).

> Information from the FoodAPS User Guide: "The National Household Food Acquisition and Purchase Survey (FoodAPS) collected information on foods purchased or otherwise acquired, and the prices and nutrient characteristics of those foods, for a nationally representative sample of U.S. households. Data on factors expected to affect food acquisition decisions, such as household size and composition, demographic characteristics, income, participation in Federal food assistance programs, and dietary restrictions, were also collected."

Files used (in `Data/CSV_data_files/`):
- `faps_household_puf.csv` — one row per household
- `faps_fahevent_puf.csv` — one row per FAH acquisition event
- `faps_access_puf.csv` — one row per household, describing the local food environment

Key variables: 
- household spending (`TOTALPAID`) and acquisition events (`EVENTID`) normalized by household size (`HHSIZE`)
- income as a percentage of the poverty line (`PCTPOVGUIDEHH_R`)
- grocery-list use (`GROCERYLISTFREQ`) 
- home meal routines (`NDINNERSOUTHH`, `NMEALSHOME`, `NMEALSTOGETHER`) 
- food security (`ADLTFSCAT`, `FOODSUFFICIENT`)
- food assistance (`SNAPNOWHH`, `WICHH`)
- transportation (`ANYVEHICLE`, `CARACCESS`)
- primary store distance and type
- local food access
- region
- rurality
- survey month

## Methodology
See the Jupyter Notebook [Capstone_Project_Final_v2.ipynb](Capstone_Project_Final_v2.ipynb) for code and analysis.

1. Data cleaning: 
   - FoodAPS missing value codes were recoded to NaN
   - Valid skips were recoded logically (ex: households with no WIC eligible member set to 0 for WIC)
   - "Meals together" and "dinners out" variables set to 0 for one person households
   - Missing event spending was imputed with the mean for the same store type
   - Missing store distance was imputed with the mean distance
   - No duplicate rows were found

2. Feature engineering: 
   - Event level data was aggregated to the household level for key variables including total spending and number of events and then merged one to one with the household and access files 
   - New features include spending and events per person, a food-insecurity flag, income as a multiple of the poverty line, season, region and vehicle access
   - The model outcome is spending per person on a log scale (log of 1 + spending), which keeps a few very large spenders from dominating the results while still allowing for households that spent $0

3. Outlier analysis: 
   - Boxplots and the standard 1.5×IQR rule (flagging values far above or below the middle 50% of households) identified a long right tail of high spenders
   - These were judged to be genuine large shopping trips and were kept
   - About 10% of households recorded $0 of FAH spending during their survey week

4. EDA: 
   - Distributions of outcomes and categorical variables, spending by grocery list use, income group, food security and home cooking, income and household size relationships and a correlation matrix

5. Baseline model: 
   - A multiple linear regression predicting log weekly FAH spending per person and a naive model that always predicts the mean

6. Modeling with grid search: 
   - Eight models were fit on the same 70/30 train/test split
   - Every model uses the same preparation steps (filling in remaining missing values with the median or most common value, putting numeric variables on the same scale, and converting categories into yes/no columns)
   - Ridge and Lasso are versions of linear regression that shrink the coefficients to avoid overfitting; Lasso can shrink unhelpful variables all the way to zero
   - GridSearchCV with 4 fold cross validation on the training data was used to try different settings for each model
   - The scoring metric was MSE (scikit-learn reports it as a negative number so that higher is better)
   - The best version of each model was then evaluated once on the test set

7. Evaluation metrics: 
   - Mean squared error (MSE), R² and cross validated MSE
   - MSE is the loss the model minimizes and penalizes large errors
   - R² shows how much of the household-to-household variation the predictors explain, which is the heart of the research question

## Results
EDA:
- Median weekly FAH spending per person rises steadily with grocery-list use and with income.
- Households that prepare more dinners at home spend more
- households with very low food security spend the least
- Spending per person falls with household size

Modeling:
- Best model: the gradient boosting model had the lowest test MSE and the highest test R², so it is the best model of those included in this study. The random forest is basically tied with it. The best model explains less than 8% of the variation in log FAH spending per person and it lowers the test MSE by only about 8% compared to always predicting the mean and about 2% compared to linear regression

- Linear models: linear, Ridge and Lasso regression perform almost the same. The linear model is not overfitting, so regularization helps very little. Lasso dropped WIC, the South region and two of the vehicle access categories, so these add little to the model.

- Polynomial models: the grid search chose degree 1, which is the same model as linear regression. Squared terms and interactions (such as income × grocery list use) raised the training fit but made the cross validated MSE worse.

- Decision tree: the best single tree is no better than linear regression on the test set. Its first splits are on income, then meals prepared at home, then household size. These are the same features the linear model found most important.

- Random forest & gradient boosting: Both fit the training data much better than the test data. In the random forest, income, distance to the primary store, meals prepared at home and household size are the most important features.

- Neural network: the grid search chose a small network with a large penalty, which might mean the more complex networks overfit. It had a slightly better cross validated MSE than linear regression

**Research question:** Are income, grocery-list use and other food planning behaviors associated with lower food-at-home (FAH) acquisition among US households?

**Answer:** they are associated with acquisition, but in the opposite direction from the hypothesis, and only modestly. The results of all the models highlight this relationship.

1. Higher income, more grocery list use, and more meals prepared at home all go with more FAH acquisition per person, not less. All the regression coefficients have the same signs. Income and meals at home are also the first splits in the decision tree and among the most important features in the random forest.

2. Eating out more is associated with lower FAH acquisition

3. Household size, food insecurity, season and region matter at least as much as the income and planning variables. Larger households spend less per person (economies of scale), food insecure households spend less, households surveyed in winter spent less (only about 200 households were surveyed in winter, all in December or January, so this estimate rests on a small group), and Western households spent more

4. More complex models improve the fit only a little. The relationships are mostly simple, and the main limitation is the data rather than the choice of model.

5. FoodAPS measures acquisition rather than waste, so one cannot make the case directly that higher spending among planners and home cooks means more waste. These households may eat more of their meals from food bought for home. A single week of acquisition data also captures a lot of noise where about 10% of households bought nothing that week, which limits how much any household characteristic can explain. Possibly better data would yield different results.

## What this means for policymakers
- Don't assume organized households waste less. The households you might expect to waste less (those who plan with grocery lists and cook at home) actually bring home the most food. Waste-reduction messaging shouldn't skip them just because they seem organized.

- Higher-income households are a reasonable first audience. They acquire more food per person, so "buy what you'll use" campaigns could start with them.

- These characteristics can't yet pick out individual high-acquisition households. Because the models explain less than 10% of the variation, better data is needed before targeting individual households, as is highlighted in the Next Steps section.

## Next Steps
1. Collect new data on higher-income households. FoodAPS deliberately oversampled SNAP participants and other low-income households, and the median household in this sample is at about 175% of the poverty line. The strongest finding is that higher income goes with more food acquisition, so the group the conclusions matter most for is the one the sample covers least. New data should cover the full income range, especially the top. It should also follow households for several weeks instead of one, which would reduce the noise from households that happened to buy nothing that week. Ideally it would measure food waste directly, such as with waste diaries, so the waste question can be answered directly instead of using acquisition as a stand-in. FoodAPS was also collected in 2012–2013, so updated data would be helpful.

2. Fit a hurdle model.The residual plots show a separate cluster of households with zero spending that a single regression cannot reach. A hurdle model splits the problem in two: a classification model for whether a household acquires any food at home during the week, then a regression for how much it acquires among households that acquired some food.

3. Try other neural networks. Deeper networks built in Keras or PyTorch, with tuned learning rates, dropout and embeddings for the categorical variables, could test whether the small network here left anything out. Since all the models performed similarly, the main purpose is to confirm the result rather than to expect large gains.

## Outline of project
- [Capstone_Project_Final_v2.ipynb](Capstone_Project_Final_v2.ipynb) — data cleaning, exploratory analysis, modeling and findings


