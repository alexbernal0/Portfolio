## Alex Bernal Portfolio

### Project 1 - Analyze Stock Returns in terms of Political Party

```markdown
Syntax highlighted code block

# Header 1
markdown cell Create a csv file with a list of all presidents, their parties from 1920 onwards.
Using Pandas load the .csv file into a Pandas dataframe.
Download daily data for S&P500 and Dow Jones Index from Yahoo! Finance.
Calculate yearly returns for both the downloaded indices from 1920 onwards.
Segregate returns in terms of Presidency – i.e. stock market returns during Democratic and Republican years.
Calculate measures of central tendency (mean return, median return, variance of returns) for each of the two groups.
Represent the findings through suitable comparative graphical studies.
Author Alex Bernal Learning Python File

# -*- coding: utf-8 -*-
"""
Created on Fri Mar 31 21:26:45 2017

@author: Alex Bernal
"""

import pandas as pd
import pandas_datareader.data as web
import os
import sys
import datetime
import numpy as np
import matplotlib.pyplot as plt


# Class to handle Presidents Data
class presidents:
    # Load data from csv
    def load_data(self, file_name):
        # Check if the file exists otherwise exit
        if os.path.isfile(file_name):
            #headers      = ['Presidency','President','Wikipedia Entry','Took office','Left office','Party','Home State']
            parse_dates  = [ 'Took office', 'Left office']
            return pd.read_csv(file_name, parse_dates=parse_dates)
        else:
            sys.exit("File {} does not exists. Exiting program".format(file_name))
    
    # Get data from returns based on party Republican/Democratic
    def get_data_for_party(self, president_data, returns, party):
        try:
            data = returns.copy(deep=True)
            for i,row in returns.iteritems():
                return_date = i
                p_data = ((president_data['Took office'] <= return_date) & (president_data['Left office'] >= return_date))
                # Delete the data inplace
                if(president_data.loc[p_data]['Party'].to_string(index=False) != party):
                    data.drop(return_date, inplace=True)
            return data
        except:
            print "Error:", sys.exc_info()[0]
            sys.exit("Error in getting data for party")
        
            

class stocks:
    # Download data from the 
    def load_data(self, ticker, start, end, interval):
        try:
            if interval == "":
                interval = 'd'
            print ("Downloading data for ticker:{} from [{},{}]".format(ticker, start, end))
            return web.get_data_yahoo(ticker, start, end)['Adj Close']
        except:
            sys.exit("Error in downloading data for ticker:{} from [{},{}]".format(ticker, start, end))
    
    def daily_returns(self, data):
        try:
            # Different ways to calculate returns
            #Regulard Daily Returns
            #return (data[1:].values - data[:-1])/data[1:]
            #Using Pandas pct_change 1 - daily, 
            #return data.pct_change(1)
            return np.log(data) - np.log(data.shift(1))
        except:
            sys.exit("Error in calculating daily returns")
            
     # Calculate YTD returns        
    def ytd_returns(self, data):
        try:
            print ("YTD Returns Rows: {}".format(len(data)))
            data_grouped = data.groupby(pd.TimeGrouper('A'))
            return data_grouped.transform(lambda x: x/x.iloc[0]-1.0)
        except:
            sys.exit("Error in calculating YTD returns")
            
    #Calculate Yearly Returns         
    def yearly_returns(self, returns):
        try:
            ret_index = (1 + returns).cumprod()
            ret_index[0] = 1
            return returns.resample('A-DEC').last().pct_change()
        except:
            sys.exit("Error in calculating yearly returns")
            
    #Calculate measures of central tendency 
    def central_tendency_measures(self, data, ticker, party):
        print "---- Stats for {}, Party {} ------".format(ticker, party)
        print "Mean : {}, Median : {}, Variance : {}".format(np.mean(data), np.nanmedian(data), np.var(data))
    
if __name__ == '__main__'    :
    default_file_name="USPresidents.csv"
    print "----------- Project 4 - Presidents-------------"
    file_name = raw_input("Enter file name. Press Enter to use default(Default : {}) :".format(default_file_name))
    if file_name == "":
        file_name = default_file_name
    
    presidents_handler = presidents()
    presidents_data = presidents_handler.load_data(file_name)
    print "Number of Presidents: {}, Columns: {}".format(len(presidents_data), list(presidents_data))
    
    print "Downloading data for S&P 500(^GSPC) and Dow Jones Index (^DJI) from Yahoo"
    tickers = ['^GSPC', '^DJI']
    stocks_helper = stocks()

    start   = datetime.datetime(1950,1,1)
    end     = datetime.datetime(2016,12,31)
    sp_data = stocks_helper.load_data(tickers[0], start, end, 'd')
    print("Ticker - {}, Rows: {}".format(tickers[0],len(sp_data)))
    sp_daily_returns = stocks_helper.daily_returns(sp_data)
    sp_yearly_returns = stocks_helper.yearly_returns(sp_daily_returns)
    
  
    start   = datetime.datetime(1985,1,1)
    dj_data = stocks_helper.load_data(tickers[0], start, end, 'd')     
    print("Ticker - {}, Rows: {}".format(tickers[1],len(dj_data)))
    dj_daily_returns = stocks_helper.daily_returns(dj_data)
    dj_yearly_returns = stocks_helper.yearly_returns(dj_daily_returns)
    
    
    rep_sp_returns = presidents_handler.get_data_for_party(presidents_data, sp_yearly_returns, 'Republican')
    rep_dj_returns = presidents_handler.get_data_for_party(presidents_data, dj_yearly_returns, 'Republican')
    
    dem_sp_returns = presidents_handler.get_data_for_party(presidents_data, sp_yearly_returns, 'Democratic')
    dem_dj_returns = presidents_handler.get_data_for_party(presidents_data, dj_yearly_returns, 'Democratic')
    
    # Plot the Graphs for Democratics and Republicans
    
    plt.plot(rep_sp_returns, color='r', label='S&P500 Returns -Republican')
    plt.plot(dem_sp_returns, color='b', label='S&P500 Returns -Democratic')
    plt.legend(loc='best')
    plt.show()
    
    plt.plot(rep_dj_returns, color='r', label='DJI Returns -Republican')
    plt.plot(dem_dj_returns, color='b', label='DJI Returns -Democratic')
    plt.legend(loc='best')
    plt.show()
    
    stocks_helper.central_tendency_measures(rep_sp_returns, 'S&P 500', 'Republican')
    stocks_helper.central_tendency_measures(dem_sp_returns, 'S&P 500', 'Democratic')
    stocks_helper.central_tendency_measures(rep_dj_returns, 'DJI', 'Republican')
    stocks_helper.central_tendency_measures(dem_dj_returns, 'DJI', 'Democratic')
    


```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Linkedin 

https://www.linkedin.com/in/alexb4/

### Contact

My contact email is alex@aetheranalytics.com
