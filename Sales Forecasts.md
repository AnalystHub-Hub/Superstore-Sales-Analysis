# Sales Forecasts for a Drugstore Chain

<p align="center"> <img src="images/rossmann.jpg"/> </p>
   
The data used is available on [Kaggle](https://www.kaggle.com/c/rossmann-store-sales). All additional information below is fictional.

## Business Problem
Rossmann is a chain of pharmacies that operates more than 3,000 stores in 7 European countries. The Rossmann CFO (Chief Financial Officer) wants to do a renovation in all of the chain's units with part of the units' revenues in the next 6 weeks.

To find out how much can be invested in the renovation of each of the units, the CFO has requested a sales prediction for each of Rossmann's units for the next 6 weeks.   

## Business Results
The model developed predicts a gross income of 285,707,584.00 USD in the next 6 weeks for the stores available, where the best and worst case scenarios results on 286,423,764.87 USD and 284,991,409.31 USD, respectively.

To make the predictions available online, the messaging application Telegram was used. In this application, the user must inform a bot created in Telegram of the ID of the store for which he wants to get the sales prediction for the next 6 weeks. The bot will then return a message with the prediciton. You can access the Telegram bot below through this [link](https://www.t.me/rossmannACL_bot).

## Business Assumptions
### Assumptions
* It was assumed that the currency is the dollar.
* Stores without information in `competition_distance` were considered to have no competition nearby.
* For stores with no information in `competition_open_since_month` and `competition_open_since_year`, it was assumed that either the store does not have a nearby competitor, or the store has a nearby competitor but it is not known when that competitor opened.
* For stores with no information in `promo2_since_week`, `promo2_since_year`, and `promo2_interval` were considered these are stores that did not participate in the promotion.

### Data Dictionary

<details>
  <summary> Description of the variables in the original data set according to <a href="https://www.kaggle.com/c/rossmann-store-sales/data">Kaggle</a>.</summary><br>

  | Variable                       | Definition
  -------------------------------- | ------------------------------------------------------------------------------------------------- |
  | `store`                        | unique ID for each store                                                                          |
  | `days_of_week`                 | weekday, starting 1 as Monday                                                                     |
  | `date`                         | date that the sales occurred                                                                      |
  | `sales`                        | amount of products or services sold in one day                                                    |
  | `customers`                    | number of customers                                                                               |
  | `open`                         | whether the store was open (1) or closed (0)                                                      |
  | `promo`                        | whether the store was participating on a promotion (1) or not (0)                                 |
  | `sate_holiday`                 | whether it was a state holiday (a = public holiday, b = easter holiday, c = christmas) or not (0) |
  | `store_type`                   | designates the store model as a, b, c or d                                                        |
  | `assortment`                   | indicates the store assorment as: a = basic, b = extra, c = extended                              |
  | `competition_distance`         | distance in meters to the nearest competitor store                                                |
  | `competition_open_since_month` | the approximate month competitor was opened                                                       |
  | `competition_open_since_year`  | the approximate year competitor was opened                                                        |
  | `promo2`                       | wheter the store was participating on a consecutive promotion (1) or not (0)                      |
  | `promo2_since_week`            | indicates the calendar week the store was participating in `promo2`                               |
  | `promo2_since_year`            | indicates the year the store was participating in `promo2`                                        |
  | `promo2_interval`              | indicates the intervals in which `promo2` started                                                 |
  </details>
  
<details>
  <summary> Description of the variables created during the project development.</summary><br>
  
  | Variable                         | Definition
  ---------------------------------- | --------------------------------------------------------------------------------------- |
  | `year`                           | year from date that the sales occurred                                                  |
  | `month`                          | month from date that the sales occurred                                                 |
  | `day`                            | day from date that the sales occurred                                                   |
  | `week_of_year`                   | week of the year from date that the sales occurred (int type)                           |
  | `year_week`                      | year and week from date that the sales occurred (object type)                           |
  | `competition_open_since`         | concatenation of `competition_open_since_year` and `competition_open_since_month`       |
  | `competition_open_timein_months` | calculates the time in months that competitor has been open based on the purchased date |
  | `promo2_since`                   | concatenation of `promo2_since_year` and `promo2_since_week`                            |
  | `promo2_since_timein_weeks`      | calculates the time in weeks that promotion began based on the purchased date           |
  | `month_map`                      | month from date that the sales occurred as auxiliar feature                             |
  | `is_promo2`                      | whether the purchase occurred during an active promo2 (1) or not (0)                    |
</details><br>

## Solution Strategy
The adopted solution is based on the CRISP-DM method (Cross Industry Standard Process for Data Mining), a cyclical and flexible methodology for solving problems involving large volumes of data that enables the rapid delivery of value to the business teams. Below is an illustration of the main steps of the solution.

<p align="center"> <img width="600" src="images/crisp_dm_based_process.png"/> </p>

As can be seen from the topics above, part of this process has already been covered: the business problem, the business understanding, and the data collection -- which in this fictitious scenario is a .csv file from a Kaggle competition, but could be a company database, a set of spreadsheets, or other sources.

## Data Preprocessing
### Data Description 
This step is important to understand how challenging the problem is in terms of computing power needed, types of variables and variable transformation needs, amount of missing data and techniques for filling them, and to have a notion of magnitudes and limits through basic statistics.

### Filling Missing Values
For this step a set of business assumptions were made (topic business assumptions). For `competition_distance` it was assumed that the nearest competitor is so far away that there is no competitor, so to fill in this missing data, a value much higher than the maximum value in `competition_distance` was used. Stores with no information in `promo2_interval` were considered these be stores that did not participate in the promotion, so the null was replaced by zero.

Already thinking about feature engineering, for `competition_open_since_month` that was null, the sale date of that row was copied to `competition_open_since_month`. There are some variables that we derive from the time that is very important for representing behavior. One of them is "for a certain event, how long ago is it that another event occurs or has occurred?". If a store don't have a close competitor, it has a certain amount of sales, when a close competition opens up, those sales tend to drop because the customers split up. When that competition matures, sales grow again but don't stay at the same level as before a competitor. The same reasoning applies to `competition_open_since_year`, `promo2_since_week`, and `promo2_since_year` -- how long has the promotion been going on when is the sale happening?

### Feature Engineering
For feature engineering, a mind map was made of what could influence sales and thus derive some features and hypotheses to get data insights from the exploratory data analysis, as well as verify the importance of these features for the machine learning model. The derived variables and their meanings can be found in Business Assumptions topic.

<p align="center"> <img width="700" src="images/mind_map.png"/> </p>

### Variable Filtering
Finally, a variable filtering was performed, removing variables considered irrelevant to the project, such as the days on which the stores were closed. Furthermore, variables that would not be available at the time of the prediction, such as the number of customers, were excluded.

## Exploratory Data Analysis
Using the mind map, hypotheses were created to be validated in the exploratory analysis. The objective was to understand how the variables impact the sales phenomenon and what is the intensity of this impact, as well as to obtain insights about the business.
Below are the top 3 insights obtained from the exploratory analysis, the rest can be found in the project notebook. 

### Top 3 Data Insights
1. Distance from competition does not have  a significant correlation with sales.
<p align="center"> <img width="800" src="images/competition_distance.png"/> </p>

2. Stores that besides continued and consecutive promotion, also are running another promotion, sell, on average, 41.70% more than stores that are running only the continued and consecutive promotion.
<p align="center"> <img src="images/onepromo_vs_bothpromo.png"/> </p>

3. In 2013 stores sold more in the second semester, but in 2014 they did not.
<p align="center"> <img width="700" src="images/semesters.png"/> </p>

## Data Modeling
Since the learning of machine learning algorithms is facilitated with numerical data that are on the same scale, Rescaling, Encoding and Transformation techniques were applied to prepare the data:

* Most of the numerical variable in the set do not have a normal distribution, rescaling methods were applied -- for variables with very strong outliers RobustScaler was used, while for the remaining variables Min-Max Scaler was used. For categorical variables Label Enconding was used.
* Since the response variable, sales, does not have a normal distribution, in order to facilitate learning the algorithms, a logarithmic transformation was applied.
* In order to respect the cyclical nature of time variables such as day, day of the week, week, and month, cyclical transformations of sine and cosine type were applied.

At the end, to increase the predictive power of the algorithms by selecting the most critical variables and eliminating redundant and irrelevant variables, the Boruta algorithm was used. 

## Machine Learning Models
### Assumptions
Given the business problem presented, it is a regression problem since the response variable must be a real value (the value of sales for each store). To solve this problem, first, the mean model was used, which is intended to serve as a baseline for comparing the performance of the algorithms.   

Since it was not yet known whether the nature of the sales phenomenon is linear or non-linear, two linear and two non-linear algorithms were selected. The algorithms selected for the test were:

* Linear Regressor
* Regularized Linear Regressor (Lasso)
* Random Forest Regressor
* XGBoost Regressor

### Time Series Cross Validation
The algorithms were evaluated using the cross validation technique, which can be illustrated in the image below.

<p align="center"> <img width="800" src="images/cross_validation.png"/> </p>

From the whole available dataset, a small portion is separated for training and another small portion for validation, and then the performance is evaluated.

In the next iteration, another portion of the data is used for training, composed of the training plus validation from the previous iteration, and another for validation that always has the same size. This is repeated successively, always respecting the chronology of the data, until the given number of iterations k.

### Algorithm Performances
By applying the Cross Validation Time Series technique to the chosen algorithms, the following results were obtained:

| Model             | MAE                 | MAPE          | RMSE               |
| ----------------- | ------------------- | ------------- | ------------------ |
| Linear Regression | 2040.76 +/- 267.47	| 0.30 +/- 0.01 | 2911.40 +/- 396.61 |
| Lasso	           | 2388.68 +/- 398.48	| 0.34 +/- 0.01 | 3369.37 +/- 567.55 |
| Random Forest	  |  837.18 +/- 220.28	| 0.12 +/- 0.02 | 1254.36 +/- 320.57 |
| XGBooster	        | 1060.91 +/- 166.35	| 0.15 +/- 0.01 | 1532.42 +/- 227.07 |

To decide the best model to use, the RMSE (Root Mean Square Error) was used. The RMSE is quite rigid, giving greater weight to larger errors and being sensitive to outliers, which makes it a good metric for measuring model performance. 

Although Random Forest and XGBoost performed better, the algorithm chosen for this first CRISP cycle was XGBoost. A Random Forest model can get very large, requiring a lot of space. This is just the nature of a Random Forest, because all the trained trees must be stored, with all their nodes. Considering that the company pays to store the models on servers, the little difference in performance may not offset the price of storage. 

To maximize the chosen model, a hyperparameter fine-tuning was performed, by random search using some sample parameters.

## Algorithm Evaluation
### Business Performance
Below, three main scenarios are compared, the actual sum of sales of all stores during the 6 weeks, the sum of the sales in the scenario where the average sales of each store is generalized for 6 weeks (mean model), and the sum of sales predicted by the machine learning model.

| Sum of Sales   | Baseline (Mean) | ML Model      |
| -------------- | --------------- | ------------- |
| 289,571,750.00 | 324,608,344.45  | 285,707,584.00|

The use of the ML model is justified when compared to using the average for projecting future revenue, since the model's deviation from the actual value was significantly lower than the baseline model (mean).

### Possible Scenarios
Based on the error calculated for the model, pessimistic and optimistic scenarios were drawn in order to give greater decision possibilities to the business team. Below are some examples of scenarios by store.

| Store	| Sales	      | Predictions	| Worst      | Best       | MAE	  | MAPE 
| ------ | ------------ | ------------ | ---------- | ---------- | ------ | ---- |
| 919	   | 207,192.00	| 205,646.70	| 205,203.00 | 206,090.41 | 443.71 | 0.08 |
| 507	   | 312,512.00	| 316,332.44	| 315,625.98 | 317,038.89 | 706.45 | 0.09 |
| 316	   | 373,108.00	| 366,507.97	| 365,819.22 | 367,196.71 | 688.74 | 0.07 |
| 983	   | 303,509.00	| 316,542.09	| 315,700.63 | 317,383.56 | 841.46 | 0.11 |

The worst and best scenarios were calculated by taking the prediction value, subtracting, and adding the MAE value. The MAE (Mean Absolute Error) tells how much the predicted data is wrong compared to the real value, in an absolute perspective. In order hand, the MAPE (Mean Absolute Percentage Error) tells about this same difference, but from a percentage perspective.

### Model Performance
An overview of the model's performance and the magnitude of its error can be seen in the chart below, where error_rate represents predictions divided by sales, and error represents sales minus predictions. 

<p align="center"> <img src="images/ml_performace.png"/> </p>

The error_rate is used to find non-random errors, especially of time-related effects. The error distribution is used to detect violation of the normality assumption. The error vs predictions - should show a random pattern of an error on either side of 0, if a point is far from most points, it may be an outlier value, and there should be no recognizable pattern in the error plot.

## Model in Production
With the model selected, trained and evaluated with a good performance, it was time to put it into production (model deployment). For this, it was decided to make the project predictions available online through the messaging application Telegram. In this application, the user must inform a bot created in Telegram the ID of the store to which he or she wants to get the sales prediction for the next 6 weeks. Thus, the bot will return a message with the prediction. 

To accomplish this task, it was necessary to create two APIs, the **handler.py**, responsible for returning the sales prediction based on the store attributes, and the **rossmann_bot.py**, responsible for communicating with the end user, managing the welcome, error, and response messages to the prediction requests. To host these applications, Heroku was used. An illustration of the process is shown below.

<p align="center"> <img width="700" src="images/schema_toget_prediction.png"/> </p>

Once the user performs a query by entering the ID of the store to which they want the prediction, the file **rossmann_bot.py** loads the attributes data of the store that are in production, performs some treatments, and transforms it into json. 

Based on this information the file **handler.py** loads the trained model, models the data and then performs the prediction. In response, the **handler.py** returns the same input dataset in json format plus an element that informs the sales prediction value for the requested store(s) and day(s).

Finally, the **rossmann_bot.py** transforms this json, sums the predictions and informs the user through a message, the total value of the sales predictions for the next 6 weeks.

## Telegram Bot
At the end of the whole process we have the production model up and running. The Telegram bot can be accessed through this [link](https://www.t.me/rossmannACL_bot).
<p align="center"> <img height="400" src="images/telegram_demo.gif"/> </p>

## References
* Introduction image from [Rosmann Media Library](https://unternehmen.rossmann.de/presse/mediathek.html).
* Dataset and variables meaning from [Kaggle](https://www.kaggle.com/c/rossmann-store-sales/overview).
* DS em Produção on [Comunidade DS](https://www.comunidadedatascience.com).
* Illustration of data store on the topic model in production by Catkuro on [Flaticon](www.flaticon.com/free-icons/database).
* Illustrations of the API's, trained model, data preparation and Telegram in the topic model in production by Kiranshastry on [Flaticon](www.flaticon.com/authors/kiranshastry).

#### 🤝 Contact me:
[![Linkedin Badge](https://img.shields.io/badge/-LinkedIn-black?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/anaclaudiarlemos//)](https://www.linkedin.com/in/anaclaudiarlemos/)
[![Gmail Badge](https://img.shields.io/badge/-Gmail-black?style=flat-square&logo=Gmail&logoColor=white&link:rlemos.anaclaudia@gmail.com)](mailto:rlemos.anaclaudia@gmail.com)
