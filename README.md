# Predicting the Price of Exchange Traded Funds With LSTM Using Tensor Flow and Keras

## <em>A model to predict the unpredictable</em>

Why it would be advantageous to be able to predict the price of stocks is almost self-evident. Many say it is not possible, with the most common rational being "if it were possible everyone would be doing it."

As much as that might make sense on the surface, I don't believe it to be a logical certainty for the following reasons:

<ol>
<li>Models currently <em>are</em> used to successfully buy and sell stocks</li> 
<li>It's very possible that the ratio of people who have the skills to make a prediction model is not high enough to affect the market</li>
<li>And Lastly, it's possible that the yield is not high enough to be advantageous to serious buyers.</li>
</ol>

However, I am never serious about anything, and if I could make something that could make \$15 a month it would still be more money than I'm making on anything else these days. Thus, I believe this is a worthwhile endeavor.

I hardly think I'm going to get rich off of this project, but I do hope that I can at least utilize Long Short Term Memory and Recurrent Neural Networks well enough to do better than a very basic model. So let's get to it.

# Data Sourcing:

Our target EFT will be VTI. To source the data history of VTI we use the [yfinance library](https://pypi.org/project/yfinance/) (thank you [Ran Aroussi!!!](https://pypi.org/user/ranaroussi/))

This is the only data needed for our ARIMA model, however to supplement it for our LSTM model we also use yfinance to collect the histories for the top 10 holdings of our ETF, as well as the DOW.

Also to supplement our LSTM model we use the monthly average of searches for 'vti' on google using google trends. To get this data visit https://trends.google.com/trends and enter 'vti'. Filter the results by date by choosing 'Custom time range' and choosing "From: '01/01/2004' To: today's date" (01/01/2004 is the earliest it will give you results for, I don't know why they allow you to select anything earlier).

Download this data, save it as 'vti_g_trends.csv' and move it into your project folder.

# Data Cleaning, Feature Selection, and Feature Engineering:

Our target data has a clear upward trend.

<p align='center'>
<img src='images/vti_trend.png'>
</p>

In order to deal with this for our ARIMA model we will difference it once. This produces a dataset that has a constant mean.

<p align='center'>
<img src='images/train_diff.png'>
</p>

For our LSTM model, there is more work to do though because there are more features to deal with. First, we need to combine all of our data.

For all of the data from yfinance we can simply concatenate the 'Close' series from each data frame together using an outer join. Because they all have identical timeseries indexes they will line up with each other nicely. The Google trend data needs to be engineered a bit though.

First, we need to move the first row to the column names. Next, we convert the 'Month' column from objects to datetimes and then set that as the index. Lastly, we convert the daily average searches using to_numeric and rename the columns.

We can't concatenate the google trend data to our dataframe like we had the other series though, because this data includes weekends. To get around that, we use a left join from our df, thus it only joins the days that our dataframe has indexes for.

Because we only care about the data related to our target we can truncate the dataframe to start when that data starts ('June 15th, 2001')

Depending on when the data is collected, google trend data is probably incomplete at the end, as it only goes till the first day of the current month. I set it to replace any nan's in the most recent 30 days with 44, which was the average for July 2020. THIS IS NOT NECESSARILY A SUSTAINABLE TECHNIQUE. But it will work for now.

Because this is older than some stocks they will not have values to begin with. We will fill those values with the stocks min value (which corresponds to their opening price).

For the time that our target predated our google trend data I filled it with the mean count, as people likely would have searched for vti had google existed back then.

Lastly, in our LSTM model, we need to scale our data. I used sklearn's RobustScaler in order to get acquainted with it.

# Modeling

In order to use our ARIMA model as a baseline to test our LSTM model against we split the data in the same way, using 90% as training and the last 10% as testing. After being trained our model's predictions achieve a Mean Absolute Error of 12.68. Visually, it is very clear that our ARIMA model was not able to deal with the effects of COVID-19

<p align='center'>
<img src='images/ARIMA_preds_vs_true.png'>
</p>

Our LSTM model did substantially better though, achieving a mean absolute error of 5.35. They say a picture is worth a thousand words, and the difference between these two says it all:

<p align='center'>
<img src='images/lstm_preds_vs_true.png'>
</p>

# Conclusion:

Our LSTM model is clearly better than our simple ARIMA model, however, is it good enough to bet on? I would say no. Considering that our current MAE is less than the average return, I can't say we'd be any better off using this model than just buying stock and doing nothing with it. The process has given me ideas for further developments though.

In the future, I would like to:

<ul>
<li>Simplify the model to a binary classification model of simply "Buy" and "Sell" and see plot predicted returns using that method.</li>
<li>Experiment with increasing the frequency of collecting stocks to compare mid-day trends rather than end of day</li>
<li>Create a user-friendly frontend and automate the collection of google trend data, so someone could run this model from a website just by knowing an ETF and it's top 10 holdings</li>
</ul>
