# Module-Challenge-7: Create a Web Application for an ETF Analyzer

We begin with our imports, in this case needing numpy, pandas, hvplot, and sqlalchemy. In this section, we also created a temporary database with a connection string, and then created an engine to interact with/confirm the contents of the database.

## Analyze a Single Assest in the FinTech ETF

### 1.
We want a query to select all of the PYPL data. 'All' can be represented with * in the query, so SELECT * FROM 'PYPL' will get the relevant data. We then create a variable, pypl_data, that will execute the query with the created engine. That data is then converted to a Pandas dataframe using the read_sql_query function as shown:

query = """

SELECT * FROM "PYPL"

""" 

pypl_data = engine.execute(query)

pypl_data_df = pd.read_sql_query(query, con=engine) 

### 2. 
To preview the dataframe, we simply use the .head() and .tail() function to see the ends of the dataframe. 

### 3. 
Now that we've reviewed the dataframe, we want to plot the daily_returns, to do so, we use the .plot() function with descriptive labels and axis x = 'time' and y = 'daily_returns', along with formatting parameters through the following code:

pypl_data_df.hvplot(figsize = (30,20), x = 'time', y = 'daily_returns', xlabel = 'Time', ylabel = 'Daily Returns', title = 'PYPL Daily Returns: 2016-12-16 - 2020-12-04')

### 4. 
To plot the cumulative returns, we first make a new daily returns dataframe, dr_df, to only get the time and the daily returns column. We set this variable to pypl_data_df.iloc[:,[0,6]], where ':' calls all rows and [0,6] gets the 'time' and 'daily_returns' column. We want the time to be the index, so we then set dr_df to itself applying .set_index('time'). After this, we want a cumulative returns dataframe, cm_df, which is set to (1 + dr_df).cumprod(), and previewed using the .tail() function. After looking at the data to check accuracy, we plot again using the .plot() function on cm_df, with label and formatting parameters included. The code goes as follows:

dr_df = pypl_data_df.iloc[:, [0, 6]]

dr_df = dr_df.set_index('time') 

cm_df = (1 + dr_df).cumprod()

cm_df.tail()

cm_df.hvplot(figsize = (30,20), title = 'PYPL Cumulative Returns: 2016-12-16 - 2020-12-04', ylabel = 'Cumulative Returns')

## Optimize SQL Queries 

### 1. 

To query just close prices above 200 from PYPL, we first SELECT 'time' and 'close' FROM 'PYPL' WHERE close > 200 in our close_query variable, we next, like before, read the close_query  using .read_sql_table(), and assigning to a variable then previewing, as shown:

close_query = """

SELECT time, close

FROM PYPL

WHERE close > 200

""" 

pypl_higher_than_200 = pd.read_sql_query(close_query, con=engine) 

pypl_higher_than_200.head() 

### 2. 

In this query, we want daily_returns in descending order of the first 10 values. To do this we SELECT both 'time' and 'daily_returns' FROM 'PYPL'. Next we use ORDER BY on 'daily_returns' followed by DESC, concluded with LIMIT 11 to get the first 11 rows of data (title row plus 10 data rows). We then read this query, descending_query, into a Pandas dataframe with .read_sql_query(), then preview the dataframe using .head(11), as shown below: 

descending_query = """

SELECT time, daily_returns

FROM PYPL 

ORDER BY daily_returns DESC

LIMIT 11

""" 

pypl_top_10_returns = pd.read_sql_query(descending_query, con=engine) 

pypl_top_10_returns.head(11) 

## Analyze the FinTech ETF Portfolio

### 1. 
To join each table for the portfolio data together, we want to SELECT 'all' with * . We also want time from the GDOT table, so we use FROM GDOT. From here for each ticker, we enter INNER JOIN [ticker] ON GDOT.time = [ticker].time, and finish the whoe query with """. We read this query, portfolio_query, into a Pandas dataframe using read_sql_table() like before. Now we take our dataframe, etf_portfolio, and assing it to itself using etf_portfolio.T.drop_duplicates().T, where .T represents transpose. In other words, this code cleans up the fact that we have multiple 'time' columns, one in each table, whcih must be condensed prior to plotting. With our etf_portfolio, we now want to modify it to follow datetime principles and set its index to 'time'. All of the following steps can be followed with:

portfolio_query = """SELECT *

FROM GDOT

INNER JOIN PYPL ON

GDOT.time = PYPL.time

INNER JOIN GS ON

GDOT.time = GS.time

INNER JOIN SQ ON

GDOT.time = SQ.time

           """

etf_portfolio = pd.read_sql_query(portfolio_query, con=engine) 

etf_portfolio = etf_portfolio.T.drop_duplicates().T


etf_portfolio 

etf_portfolio['time'] = pd.to_datetime(etf_portfolio['time']).dt.date

etf_portfolio = etf_portfolio.set_index('time')

print(etf_portfolio)

### 2. 
For average daily returns of the dataframe, we assign etf_portfolio['daily_returns'].mean(axis=1) to a etf_portfolio_returns variable, then preview.

### 3. 
For annualized returns, we will crete a variable and assign it to etf_portfolio_returns.mean() * 252, representing the number of trading days. This is followed by a print statement with an f-string to display the annualized return in terms of percent, as shown:

annualized_etf_portfolio_returns =  (etf_portfolio_returns.mean() * 252)

print(f"The annualized return value of the ETF portfolio is: {annualized_etf_portfolio_returns * 100: .2f}%")

### 4. 
For cumulative returns of the whole etf, we create a variable and assign it to (1 + etf_portfolio_returns).cumprod(), followed by a similar print statement to (3, with the code as follows:

etf_cumulative_returns = (1+etf_portfolio_returns).cumprod()

print(f"The final cumulative return value is: {etf_cumulative_returns.tail(1)} times the initial value")

### 5. 
Lastly, using the .hvplot() function on the etf_cumulative_returns dataframe, we create an interactive plot of the cumulative returns, with x = 'time', and other parameters pertaining to labeling and formatting, which can be seen below:

etf_cumulative_returns.hvplot(x="time",

                              xlabel="Time",
                              
                              ylabel="Cumulative Return Value",
                              
                              title="ETF Portfolio Cumulative Returns Plot 2016-2020",
                              
                              height=500,
                              
                              width=1000)
