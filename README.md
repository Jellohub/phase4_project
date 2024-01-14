# FLATIRON SCHOOL PHASE 4 PROJECT
*Angelo Turri*
_
*angelo.turri@gmail.com*

## Stakeholder & Problem
Our stakeholder is a home-flipper in Michigan. Their aim is to buy properties, keep them for several months, renovate them, and re-sell them for a profit. They have provided us with data from across the US; they want us to select the best 5 Michigan zip-codes to invest in.

## Data Origin & Description
The dataset in this project was taken from [this page](https://www.zillow.com/research/data/) and stored on [this github repository](https://github.com/learn-co-curriculum/dsc-phase-4-choosing-a-dataset/tree/main/time-series) as "zillow_data.csv."

After importing and adjusting the dataset so it could be used for modeling, it contained 265 months of data from April 1996 – April 2018 for 13,684 zipcodes, totalling 3.6 million rows of data.

We condensed the dataset down to just houses in Michigan, and then we condensed it further to houses sold during and after 2012. We chose this time period because it was only during 2012 that the housing market came out of the decline caused by the 2008 housing market crash. We didn't want data from a rare, unlikely event to impact our models.

After this preprocessing, our data included 76 months of data for 453 zipcodes, totalling roughly 34,000 rows of data. Each row represents the average house price in a zipcode on the first day of a particular month.

#### Price trend including 2008 market crash
![Price trend including 2008 market crash](/figures/trend_before.png)

#### Price trend from 2012 and after
![Price trend from 2012 and after](/figures/trend_after.png)

## Methods Justification & Value to Stakeholder
Our goal was to give our stakeholders the five zipcodes offering the highest ROI (return on investment) while not having volatile house prices. The idea is to provide low-risk, good-reward opportunities for investment.

We use an **iterative modeling process**, meaning we start with a baseline model and make changes to our model based on model performance.
    
## Limitations
Data before 2012 includes anomalous and low-probability events that can adversely affect model performance (e.g., the 2008 housing market crash). After removing this data, the amount of resulting data is smaller than ideal (76 months of data per zip code).

## Metrics
In this project, we used several metrics to evaluate model performance and certain attributes of our dataset. Our model metrics included:

- RMSE: root mean squared error. This is the average number of dollars that each model prediction was wrong by.
- Two tests for stationarity performed on model residuals. These tests determine whether or not the model reduced its dataset to a stationary one. If it did, that means it successfully extracted all temporal components from that dataset. We used two stationarity tests becasue two such tests in agreement is more powerful evidence of stationarity than a single such test saying so.
    - Augmented dickey-fuller test
    - Kwiatkowski–Phillips–Schmidt–Shin (KPSS) test
    
    
We used ROI after three months as a way of measuring the profitability of an investment. ROI is the projected percentage increase of an investment after a certain time period. So if our ROI in a zipcode is 1.2, that means that our investment is projected to grow by 20% over the time period in question (in this case, three months).
    
We needed a way of measuring volatility for each zipcode's price over time. Looking at each zipcode manually is time-consuming; ideally, we would have a way of scoring each zipcode's volatility. I developed a score for measuring volatility that involves measuring the magnitude of fluctuations in price and normalizing them with the range of values seen in that zipcode's price. Here were the steps for measuring that:
- Take the approximate values of the second derivative for each zipcode's price
    - The second derivative measures the change in slope.
    - If a fluctuation is more severe, the magnitude of the second derivative during the fluctuation will be higher.
    - The more the price fluctuates, the more often its second derivative will have a high value.
- Square these values to stop negative values from cancelling out positive ones
- Take the average of these values to get a sense of the average level of price fluctuation throughout the time period
- Normalize this average with the range of prices throughout the time period.
    - The range is used because it brings the size of the fluctuations into perspective.
    
If this volatility function really works, we should expect several things to happen:

- As a zipcode's volatility score goes up, the price action will fluctuate more;
- As a zipcode's volatility score goes up, the expected ROI will become more unpredictable.

Both of the visualizations below confirm this. We see that zipcodes become more and more volatile the higher their volatility score, and as the volatility score increases, the distribution of their ROIs becomes more uniform.
![Prices of zipcodes with different volatility scores](/figures/volatility_graphs.png)
![Distributions of ROI within zipcodes of different volatility scores](/figures/volatility_distributions.png)


## Modeling
Our baseline model was a naive model, a good starting point for developing good time-series models. The naive model simply shifts all values forward by one chosen time unit (in this case a month). Each shifted value becomes the model's "prediction" for that month, and its one-month forecast is just the latest recorded value.

Our next model was an ARIMA model with an order of (1,1,1). The parameters for this model were informed by ACF (autocorrelation function) and PACF (partial autocorrelation function) plots for the Michigan sales data. This model's performance increased on the training data. However, overfitting also increased.

In our next round of modeling, we improved model performance by using a rolling forecast instead. A rolling forecast makes a prediction, appends that prediction to its training data, and re-trains itself on the updated training data to make its next prediction. Using a rolling forecast slightly improved model performance by decreasing the amount of overfitting.

For our final model, we threw out the train-test-split, as this split was only used to gauge overfitting. 
When making your final predictions, you want to use all the data available to you. Because the rolling ARIMA forecast was the least overfitting model, we trained that model on all our available time series data and used it to make predictions 3 months into the future.

## Recommendations

Utilizing the scoring method I mentioned earlier, I narrowed down Michigan's zipcodes to a list of low-volatility zipcodes (volatility score <5), and from that list, I selected the zipcodes that yielded the highest ROI. These zipcodes were recommended to the stakeholder. They are:
![All five recommended zipcodes](/figures/all_five.png)

Here are the individual graphs and projected forecasts for each zipcode:

![Zip #48240](/figures/zip1.png)
![Zip #48184](/figures/zip2.png)
![Zip #48237](/figures/zip3.png)
![Zip #48033](/figures/zip4.png)
![Zip #48239](/figures/zip5.png)

Here are the 95% confidence intervals for each investment. The true ROI is 95% likely to be within the projected ROI, plus or minud 1%. We got this number from the standard deviation of ROI within zipcodes of a volatility score <5; we know that 95% of all objects in a normal distribution are within 2 standard deviations of the mean, and ROI was approximately normally distributed among zipcodes with a volatility score <5.
![95% confidence intervals](/figures/CIs.png)

## Repository Structure
- **[data](https://github.com/Jellohub/phase4_project/tree/master/data)**: folder containing our only data file.
    - zillow_data.csv: time series data for 13,684 zipcodes in wide format
- **[figures](https://github.com/Jellohub/phase4_project/tree/master/figures)**: folder containing all visualizations used in the project notebook and presentation.
- **[project notebook](https://github.com/Jellohub/phase4_project/blob/master/notebook.ipynb)**: project notebook containing all data imports, preprocessing, analyses, models, and visualizations.
- **[project notebook pdf](https://github.com/Jellohub/phase4_project/blob/master/notebook.pdf)**: pdf version of project notebook.
- **[presentation pdf](https://github.com/Jellohub/phase4_project/blob/master/presentation.pdf)**: pdf file with all presentation slides.