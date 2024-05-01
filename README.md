# Market-Cap-Weighted-Index-
In finance, market indices serve as vital benchmarks for measuring the performance of various sectors and the broader market as a whole. One such significant index is the market-cap weighted index, which provides investors with insights into the collective performance of stocks based on their market capitalization. This project focuses on constructing a market-cap weighted index by selecting stocks with the highest market capitalization from diverse sectors. The goal is to create a comprehensive index that accurately reflects the overall market trends and dynamics.

#### Index Construction

The index is constructed by meticulously selecting stocks from different sectors, ensuring representation across various industries. Stocks with the highest market capitalization within each sector are chosen to form the components of the index. This selection process aims to capture the most influential and impactful companies within their respective sectors, thereby creating a well-rounded index portfolio.

#### Data Collection and Analysis

To facilitate the construction of the index, historical stock data is collected and analyzed using the Yahoo Finance API. This data includes essential metrics such as stock prices, market capitalization, and sector information. By leveraging this data, the project ensures a robust and data-driven approach to index construction.

#### Evaluation and Comparison

Once the index components are selected and historical data is gathered, the project evaluates the performance of the market-cap weighted index over a five-year period. Comparative analysis with benchmark indices like the S&P 500 provides valuable insights into the index's performance relative to the broader market.

#### Insights and Implications

By analyzing the index's performance, investors can gain valuable insights into market trends, sector dynamics, and investment opportunities. The construction of a market-cap weighted index allows for a comprehensive evaluation of market performance, aiding investors in making informed decisions and optimizing their investment strategies.

## Data Collection, Loading and Cleaning

This code retrieves information about stocks using their ticker symbols from different stock exchanges (NYSE, NASDAQ, and AMEX), stores the data in a DataFrame, and then saves it to a CSV file. This code essentially automates the process of collecting stock information from multiple sources, consolidates it into a single dataset, and saves it for further analysis.
``` py

def get_stock_info(ticker): # This function takes a ticker symbol as input.
    try:
        stock = yf.Ticker(ticker) #It uses the yfinance library to retrieve information about the stock, such as sector and market capitalization
        info = stock.info
        
        sector = info.get('sector')
        market_cap = info.get('marketCap')
        
        return sector, market_cap
    except:
        return None, None #If the information is successfully retrieved, it returns the sector, market capitalization, and IPO date. If not, it returns None.

# Reading ticker symbols from CSV files.
nyse_df = pd.read_csv(r'nyse.csv')  
nasdaq_df = pd.read_csv(r'nasdaq.csv')  
amex_df = pd.read_csv(r'amex.csv')  

# Extracting ticker symbols
nyse_tickers = nyse_df['Symbol'].tolist()
nasdaq_tickers = nasdaq_df['Symbol'].tolist()
amex_tickers = amex_df['Symbol'].tolist()

# Combining all tickers
all_tickers = nyse_tickers + nasdaq_tickers + amex_tickers
data = []

# Getting stock information for each ticker and store it in the list
for ticker in all_tickers:
    sector, market_cap, ipo_date = get_stock_info(ticker)
    data.append([ticker, sector, market_cap, ipo_date])

# Create a DataFrame from the data list
df = pd.DataFrame(data, columns=['Ticker', 'Sector', 'Market Cap', 'IPO Date'])
df.head()
df.info()
# Save the DataFrame to a CSV file
df.to_csv('stock_info.csv', index=False)
```
This code segment loads, cleans, and combines stock listing data from a CSV file which we automated in the prevoius section. Overall, this code segment loads stock listing data from a CSV file, cleans it by removing rows with missing sector and market capitalization information, and prepares it for further analysis.
``` py
listings = pd.read_csv("stock_info.csv", index_col = 'Ticker') # creating Excel file object
listings.head()
listings.info()
listings.dropna(subset=["Sector"], inplace=True)
listings.dropna(subset=["Market Cap"], inplace=True)
listings.head()
listings.info()
```

```
                    Sector    Market Cap
Ticker                                  
A               Healthcare  4.016026e+10
AA         Basic Materials  6.309704e+09
AACT    Financial Services  6.637500e+08
AAN            Industrials  2.116222e+08
AAP      Consumer Cyclical  4.350302e+09
```
## Selecting Index Components:
Here, we select the required tickers for a market cap-weighted index from each sector. Identifies the leading stocks from each sector based on their market capitalization, which are then chosen as the components of the market cap-weighted index.
``` py
categorical = listings.groupby(["Sector"])["Market Cap"].nlargest(1)
print(categorical)
categorical.info()
index_tickers = categorical.index.get_level_values("Ticker").tolist()
print(index_tickers)
```
```
['LIN', 'GOOG', 'AMZN', 'WMT', 'XOM', 'JPM', 'LLY', 'GE', 'PLD', 'MSFT', 'NEE']
```
Here's a description of each ticker included in the index along with their corresponding company names and sectors:

1. **LIN (Linde plc):**
   - Company Name: Linde plc
   - Sector: Industrial Gases
   
2. **GOOG (Alphabet Inc. - Class C):**
   - Company Name: Alphabet Inc.
   - Sector: Technology (Internet Services & Infrastructure)

3. **AMZN (Amazon.com Inc.):**
   - Company Name: Amazon.com Inc.
   - Sector: Consumer Discretionary (Internet & Direct Marketing Retail)

4. **WMT (Walmart Inc.):**
   - Company Name: Walmart Inc.
   - Sector: Consumer Staples (Hypermarkets & Super Centers)

5. **XOM (Exxon Mobil Corporation):**
   - Company Name: Exxon Mobil Corporation
   - Sector: Energy (Integrated Oil & Gas)

6. **JPM (JPMorgan Chase & Co.):**
   - Company Name: JPMorgan Chase & Co.
   - Sector: Financials (Diversified Banks)

7. **LLY (Eli Lilly and Company):**
   - Company Name: Eli Lilly and Company
   - Sector: Healthcare (Pharmaceuticals)

8. **GE (General Electric Company):**
   - Company Name: General Electric Company
   - Sector: Industrials (Industrial Conglomerates)

9. **PLD (Prologis Inc.):**
   - Company Name: Prologis Inc.
   - Sector: Real Estate (Industrial REITs)

10. **MSFT (Microsoft Corporation):**
    - Company Name: Microsoft Corporation
    - Sector: Technology (Systems Software)

11. **NEE (NextEra Energy Inc.):**
    - Company Name: NextEra Energy Inc.
    - Sector: Utilities (Electric Utilities)

These companies represent a diverse range of sectors, including technology, retail, energy, finance, healthcare, industrials, real estate, and utilities, providing broad exposure to the market.
## Fetching Historical Stock Data:
Using Yahoo Finance API (yfinance), the code fetches historical stock data for the selected index components starting from January 1, 2019. This code segment fetches historical stock data for the selected index components, calculates the latest prices of these components, and then calculates the number of outstanding shares for each component.
``` py
start = date(2019,1,1)
stock_data = pdr.get_data_yahoo(index_tickers, start) # Fetching historical data of index component
stock_data.info()
stock_data.head()
print(stock_data)
components = listings.loc[index_tickers] # Creating Dataframe containing component info
components.drop_duplicates(inplace=True)
components.reset_index(inplace=True)
components.info()
components.head()

latest_date = stock_data.index.max()
latest_prices = stock_data.loc[latest_date, ('Open', slice(None))]

components['Price'] = latest_prices.values
components.set_index("Ticker", inplace=True)
components.info()
components.head()
shares = components['Market Cap'].div(components['Price'])
shares_df = shares.to_frame(name='Shares_outstanding')
shares_df.set_index(components.index, inplace=True)
print(shares_df)
print(components)
```
```
stock_data.info()
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 1342 entries, 2019-01-02 to 2024-05-01
Data columns (total 66 columns):
 #   Column             Non-Null Count  Dtype  
---  ------             --------------  -----  
 0   (Adj Close, AMZN)  1342 non-null   float64
 1   (Adj Close, GE)    1342 non-null   float64
 2   (Adj Close, GOOG)  1342 non-null   float64
 3   (Adj Close, JPM)   1342 non-null   float64
 4   (Adj Close, LIN)   1342 non-null   float64
 5   (Adj Close, LLY)   1342 non-null   float64
 6   (Adj Close, MSFT)  1342 non-null   float64
 7   (Adj Close, NEE)   1342 non-null   float64
 8   (Adj Close, PLD)   1341 non-null   float64
 9   (Adj Close, WMT)   1342 non-null   float64
 10  (Adj Close, XOM)   1342 non-null   float64
 11  (Close, AMZN)      1342 non-null   float64
 12  (Close, GE)        1342 non-null   float64
 13  (Close, GOOG)      1342 non-null   float64
 14  (Close, JPM)       1342 non-null   float64
 15  (Close, LIN)       1342 non-null   float64
 16  (Close, LLY)       1342 non-null   float64
 17  (Close, MSFT)      1342 non-null   float64
 18  (Close, NEE)       1342 non-null   float64
 19  (Close, PLD)       1341 non-null   float64
 20  (Close, WMT)       1342 non-null   float64
 21  (Close, XOM)       1342 non-null   float64
 22  (High, AMZN)       1342 non-null   float64
 23  (High, GE)         1342 non-null   float64
 24  (High, GOOG)       1342 non-null   float64
 25  (High, JPM)        1342 non-null   float64
 26  (High, LIN)        1342 non-null   float64
 27  (High, LLY)        1342 non-null   float64
 28  (High, MSFT)       1342 non-null   float64
 29  (High, NEE)        1342 non-null   float64
 30  (High, PLD)        1341 non-null   float64
 31  (High, WMT)        1342 non-null   float64
 32  (High, XOM)        1342 non-null   float64
 33  (Low, AMZN)        1342 non-null   float64
 34  (Low, GE)          1342 non-null   float64
 35  (Low, GOOG)        1342 non-null   float64
 36  (Low, JPM)         1342 non-null   float64
 37  (Low, LIN)         1342 non-null   float64
 38  (Low, LLY)         1342 non-null   float64
 39  (Low, MSFT)        1342 non-null   float64
 40  (Low, NEE)         1342 non-null   float64
 41  (Low, PLD)         1341 non-null   float64
 42  (Low, WMT)         1342 non-null   float64
 43  (Low, XOM)         1342 non-null   float64
 44  (Open, AMZN)       1342 non-null   float64
 45  (Open, GE)         1342 non-null   float64
 46  (Open, GOOG)       1342 non-null   float64
 47  (Open, JPM)        1342 non-null   float64
 48  (Open, LIN)        1342 non-null   float64
 49  (Open, LLY)        1342 non-null   float64
 50  (Open, MSFT)       1342 non-null   float64
 51  (Open, NEE)        1342 non-null   float64
 52  (Open, PLD)        1341 non-null   float64
 53  (Open, WMT)        1342 non-null   float64
 54  (Open, XOM)        1342 non-null   float64
 55  (Volume, AMZN)     1342 non-null   int64  
 56  (Volume, GE)       1342 non-null   int64  
 57  (Volume, GOOG)     1342 non-null   int64  
 58  (Volume, JPM)      1342 non-null   int64  
 59  (Volume, LIN)      1342 non-null   int64  
 60  (Volume, LLY)      1342 non-null   int64  
 61  (Volume, MSFT)     1342 non-null   int64  
 62  (Volume, NEE)      1342 non-null   int64  
 63  (Volume, PLD)      1341 non-null   float64
 64  (Volume, WMT)      1342 non-null   int64  
 65  (Volume, XOM)      1342 non-null   int64  
dtypes: float64(56), int64(10)
memory usage: 734.7 KB
```
Finally, the creation of the ```components``` DataFrame allows for efficient organization and access to the relevant information required for constructing and analyzing the market cap-weighted index. It streamlines the data preparation process and ensures that only the necessary data is retained, enhancing the clarity and effectiveness of subsequent analyses.
```
                        Sector    Market Cap       Price
Ticker                                                  
LIN            Basic Materials  2.120000e+11  181.889999
GOOG    Communication Services  2.020000e+12  162.570007
AMZN         Consumer Cyclical  1.820000e+12  166.000000
WMT         Consumer Defensive  4.780000e+11  192.270004
XOM                     Energy  4.660000e+11  442.790009
```
## Calculating Market Cap and Index Performance:
This code segment calculates the aggregated market capitalization and performance of the market cap-weighted index based on historical stock data and the number of outstanding shares. The code calculates market capitalization for each index component based on the latest available prices. It then calculates the historical performance of the index by aggregating market capitalizations and normalizing to a base value of 100. 
``` py
historical_prices = stock_data["Adj Close"]
historical_prices.info()
print(historical_prices)
market_cap_series = historical_prices.mul(shares_df['Shares_outstanding'], axis=1)
print(market_cap_series)
# Aggregated market capitalization
agg_mcap=market_cap_series.sum(axis=1)
print(agg_mcap)
index = agg_mcap.div(agg_mcap.iloc[0]).mul(100)
index.info()
index.head()
print(index)
```
## Visualizing Index Performance:
The index performance is visualized using matplotlib with a white grid style and a specified background color. The plot shows the trend of the market-cap weighted index over time.
``` py
sns.set_style("whitegrid")
fig = plt.figure(figsize=(10, 6), dpi=300).set_facecolor('#FEFDED')   # Set background color using hexcode
index.plot(color='#76ABAE', linewidth=2)  # Using hexcode for blue color
plt.title('Market-cap Weighted Index', fontsize=16)
plt.xlabel('Date', fontsize=12)
plt.ylabel('Index Value', fontsize=12)
plt.grid(True,)
plt.legend(['Index'], loc='upper left')
plt.axhline(y=100, color='#31363F', linestyle='--')
plt.show()
```
#### Visualization
![Market-cap Weighted Index](https://github.com/iftekhar-kabir/Market-Cap-Weighted-Index-/assets/163831745/d44e0d7c-f7f9-41b7-86f8-5c82aa18b37a)

## Further Evaluation
The code evaluates component weights, value-weighted component returns, and compares the index performance with the S&P 500 index. Component weights provide insights into the relative importance of each stock in the index, helping investors understand which stocks have a larger impact on the index's performance. The index return represents the overall performance of the market cap-weighted index over the specified period, serving as a key metric for evaluating investment performance. Value-weighted component returns indicate how much each index component contributes to the index's total return, reflecting the impact of individual stock performance on the index's overall performance.
``` py

# Component weights
components.head()
market_cap = components['Market Cap']
weights = market_cap.div(market_cap.sum())
weights.sort_values().mul(100)
print(weights) 

# Value-weighted component returns
index_return = (index.iloc[-1] / index.iloc[0] - 1) * 100
print(index_return) # Index Return of 5 years

weighted_return = weights.mul(index_return) # Value-weighted component returns
print(weighted_return)
```
#### Visualization
![Value_weighted Component Returns](https://github.com/iftekhar-kabir/Market-Cap-Weighted-Index-/assets/163831745/61f043a7-2324-4a6d-8e3a-cf326bee546f)
``` py
sns.set_style("whitegrid")
plt.figure(figsize=(10, 6), dpi=300, facecolor='#FEFDED')
weighted_return.sort_values().plot(kind='barh', color='#76ABAE')
plt.title('Value-weighted Component Returns', fontsize=16)
plt.xlabel('Index Value', fontsize=12)
plt.ylabel('Ticker', fontsize=12)
plt.grid(True)
plt.show()
```
## Index VS S&P 500
This code segment compares the performance of the market cap-weighted index with the performance of the S&P 500 index over a specified period. This code segment allows for a comparative analysis of the performance of the market cap-weighted index and the S&P 500 index over the specified period. By normalizing both indices to a common starting point, it facilitates a meaningful comparison of their performance trends and relative returns.
``` py
df_market = index.to_frame("Index")
SP500 = pd.read_csv("G:\Downloads\S&P 500 Historical Data.csv", 
                     parse_dates = ['Date'], index_col= 'Date')
SP500.drop(columns =['Open', 'High', 'Low', 'Vol.', 'Change %'], inplace=True)
SP500["Price"] = SP500.Price.str.replace(",","")
SP500["Price"] = SP500.Price.astype(float)
SP500.info()
df_market["SP500"] = SP500.Price
df_market.SP500 = df_market.SP500.div(df_market.SP500.iloc[0], axis=0).mul(100)
df_market.head()
```
#### Visualization
![image](https://github.com/iftekhar-kabir/Market-Cap-Weighted-Index-/assets/163831745/8408154a-68ce-4df8-b4e7-6dbe1b85cf79)
``` py
sns.set_style("whitegrid")
plt.figure(figsize=(10, 6), dpi=300, facecolor='#FEFDED')
df_market["Index"].plot(color='#76ABAE', linewidth=2)  
df_market["SP500"].plot(color='#FFA500', linewidth=2)  
plt.title('Market-cap Weighted Index vs S&P 500', fontsize=16)
plt.xlabel('Date', fontsize=12)
plt.ylabel('Index Value', fontsize=12)
plt.grid(True)
plt.legend(['Index', 'S&P 500'], loc='upper left')
plt.axhline(y=100, color='#31363F', linestyle='--')
plt.show()
```
## Volatility:
This code segment calculates the rolling volatility of the market cap-weighted index and the S&P 500 index over a 30-day period. Rolling volatility is a measure of the variability of returns over a specified period, providing insights into the market's stability and risk levels over time. By calculating multi-period returns over a rolling window, this code segment captures the changing volatility of the market cap-weighted index and the S&P 500 index, allowing for a dynamic assessment of their risk levels.
#### Visualization
![image](https://github.com/iftekhar-kabir/Market-Cap-Weighted-Index-/assets/163831745/fdeaebdb-a61e-45ef-a8be-23089e81c080)
``` py
sns.set_style("whitegrid")
plt.figure(figsize=(10, 6), dpi=300, facecolor='#FEFDED')
multi_period_returns["Index"].plot(color='#76ABAE', linewidth=1.5) 
multi_period_returns["SP500"].plot(color='#FFA500', linewidth=1)
plt.title('30 Days Rolling Avg: Market-cap Weighted Index vs S&P 500', fontsize=16)
plt.xlabel('Date', fontsize=12)
plt.ylabel('Return (%)', fontsize=12)
plt.grid(True)
plt.legend(['Index', 'S&P 500'], loc='upper left')
plt.rcParams['figure.dpi'] = 300
plt.show()
```
## Correlation of Stocks:
This code segment calculates the daily returns of the historical stock prices and then computes the correlation matrix of these returns. Daily returns capture the day-to-day price fluctuations of the index components, providing insights into their short-term performance and volatility. The correlation matrix helps identify relationships between index components' returns, highlighting whether they move in the same direction (positive correlation), opposite directions (negative correlation), or independently (zero correlation). Understanding the correlations between index components' returns is crucial for diversification strategies, risk management, and portfolio construction, as it informs investors about the degree of interdependence among assets within the index.
#### Visualization
![image](https://github.com/iftekhar-kabir/Market-Cap-Weighted-Index-/assets/163831745/dbb750bb-10a2-4d4f-9e7f-bcb935f3254d)
``` py
sns.heatmap(correlations, annot=True)
plt.xticks(rotation=45)
plt.title('Daily Return Correlations')
```
## Performance Evaluation Summary

1. **Data Preparation and Analysis:**
   - The code begins by loading and cleaning stock listing data from multiple Excel worksheets.
   - It selects the required tickers for the market cap-weighted index from each sector based on their market capitalization.
   - Historical data for the selected index components is fetched from Yahoo Finance using the `pdr.get_data_yahoo()` function.
   - The fetched data is then used to calculate various metrics, such as market capitalization, component prices, and shares outstanding.

2. **Index Performance:**
   - The historical performance of the market-cap weighted index is calculated based on the aggregated market capitalization of its components.
   - The performance is visualized using a line plot, providing insights into the index's trend and fluctuations over time.

3. **Component Analysis:**
   - Component weights are determined based on their market capitalization relative to the total market capitalization of all components.
   - Value-weighted returns of individual components are calculated and visualized using a horizontal bar plot, indicating the contribution of each component to the index's overall performance.

4. **Comparison with S&P 500:**
   - The performance of the market-cap weighted index is compared with that of the S&P 500 index.
   - Both indices' performance is visualized on the same plot, allowing for direct comparison of their trends and movements over time.

5. **Volatility and Correlation Analysis:**
   - The volatility of the market-cap weighted index and the S&P 500 index is analyzed using rolling average returns over a 30-day period.
   - Correlation analysis is conducted to examine the relationship between daily returns of the index components.

6. **Overall Evaluation:**
   - Through visualizations and analysis, the code provides insights into the market-cap weighted index's performance relative to the broader market represented by the S&P 500 index.
   - By considering various metrics and conducting comparative analysis, the code offers a comprehensive evaluation of the index's performance over the last five years.

The evaluation suggests that the market-cap weighted index has exhibited certain trends, fluctuations, and correlations over the analyzed period, providing valuable insights for investors and analysts.

## References
1. https://github.com/rreichel3/US-Stock-Symbols
2. https://finance.yahoo.com/
3. https://www.investing.com/indices/us-spx-500-historical-data
4. https://stackoverflow.com/
5. https://www.w3schools.com/
