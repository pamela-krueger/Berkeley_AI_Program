# Comparing Classifiers  
  
Code can be found in the Jupyter Notebook called ‘prompt_III_final_solution.ipynb’  
  
## Business Objective  
  
Marketing campaigns for this bank's term deposit product have become less effective over time as such campaigns have become ubiquitous industry-wide (and thus not very useful). The bank wants to identify which client attributes predict a "yes" subscription so that future phone campaigns can be targeted at the clients most likely to subscribe, reducing wasted calls and campaign cost.  
  
## Modeling  
  
Logistic Regression, K-Nearest Neighbors, Decision Tree, and SVM were compared. A baseline was established first, then each model was fit with default settings. The final models were tuned using GridSearchCV using ROC AUC since accuracy is a misleading metric.  
  
## Final Results  
  
**After hyper-parameter tuning**  

| Model               | Test ROC AUC | Precision | Recall | F1   |
| ------------------- | ------------ | --------- | ------ | ---- |
| Logistic Regression | 0.7933       | 0.31      | 0.69   | 0.43 |
| KNN                 | 0.7590       | 0.67      | 0.20   | 0.31 |
| Decision Tree       | 0.7957       | 0.67      | 0.25   | 0.37 |
| SVM                 | 0.7752       | 0.36      | 0.66   | 0.46 |
  
  
  
## Recommendation  
  
Logistic Regression is the recommended model with a comparable recall to SVM but with much less training time.  
