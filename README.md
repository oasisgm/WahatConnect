Trading Application Using Zerodha's Kite Connect API
This project is a Python-based trading application that leverages Zerodha's Kite Connect API to fetch historical stock data, compute technical indicators, generate trade signals, and execute trades automatically.

Features
Authentication: Securely connect to the Kite Connect API using your API key and secret.
Historical Data Retrieval: Fetch historical data for specified instruments.
Technical Indicators: Calculate indicators like RSI and MACD using TA-Lib.
Trade Signal Generation: Generate buy or sell signals based on indicator values.
Automated Trading: Place orders automatically based on generated signals.
Prerequisites
Python 3.6 or higher
Zerodha Kite Connect API subscription
Zerodha trading account
Installation
Clone the Repository:

bash
Copy
Edit
git clone https://github.com/yourusername/zerodha-trading-app.git
cd zerodha-trading-app
Install Dependencies:

bash
Copy
Edit
pip install -r requirements.txt
Ensure that the requirements.txt file includes the following packages:

kiteconnect
pandas
ta-lib
openai
speechrecognition
Note: TA-Lib requires additional system dependencies. Refer to the TA-Lib installation guide for detailed instructions.

Setup
Kite Connect API Credentials:

Sign up on the Kite Trade website and create a new app to obtain your API_KEY and API_SECRET.
Authentication:

Run the authentication script to generate and set the access token:

python
Copy
Edit
from kiteconnect import KiteConnect

kite = KiteConnect(api_key="your_api_key")
print(kite.login_url())
Visit the URL printed by the script, authorize the app, and obtain the request_token.

Exchange the request_token for an access_token:

python
Copy
Edit
data = kite.generate_session("your_request_token", api_secret="your_api_secret")
kite.set_access_token(data["access_token"])
Store the access_token securely for future use.

Usage
Fetch Historical Data:

python
Copy
Edit
from datetime import datetime, timedelta

instrument_token = 738561  # Example: Instrument token for RELIANCE
start_date = datetime.now() - timedelta(days=30)
end_date = datetime.now()

historical_data = kite.historical_data(
    instrument_token,
    start_date,
    end_date,
    interval="day"
)
Calculate Technical Indicators:

python
Copy
Edit
import talib
import pandas as pd

df = pd.DataFrame(historical_data)
df['RSI'] = talib.RSI(df['close'], timeperiod=14)
df['MACD'], df['MACD_signal'], _ = talib.MACD(df['close'])
Generate Trade Signals:

python
Copy
Edit
def generate_signal(data):
    if data['RSI'].iloc[-1] < 30 and data['MACD'].iloc[-1] > data['MACD_signal'].iloc[-1]:
        return "BUY"
    elif data['RSI'].iloc[-1] > 70 and data['MACD'].iloc[-1] < data['MACD_signal'].iloc[-1]:
        return "SELL"
    else:
        return "HOLD"

signal = generate_signal(df)
print(f"Trade Signal: {signal}")
Execute Trades:

python
Copy
Edit
if signal == "BUY":
    order_id = kite.place_order(
        variety=kite.VARIETY_REGULAR,
        exchange=kite.EXCHANGE_NSE,
        tradingsymbol="RELIANCE",
        transaction_type=kite.TRANSACTION_TYPE_BUY,
        quantity=1,
        order_type=kite.ORDER_TYPE_MARKET,
        product=kite.PRODUCT_CNC
    )
    print(f"Buy order placed. ID: {order_id}")
elif signal == "SELL":
    order_id = kite.place_order(
        variety=kite.VARIETY_REGULAR,
        exchange=kite.EXCHANGE_NSE,
        tradingsymbol="RELIANCE",
        transaction_type=kite.TRANSACTION_TYPE_SELL,
        quantity=1,
        order_type=kite.ORDER_TYPE_MARKET,
        product=kite.PRODUCT_CNC
    )
    print(f"Sell order placed. ID: {order_id}")
else:
    print("No trade action taken.")
Important Notes
Manual Authentication: Due to regulatory requirements, the initial authentication requires manual intervention to obtain the request_token.

Error Handling: Implement appropriate error handling to manage exceptions during API calls.

Testing: Thoroughly test the application in a simulated environment before deploying it for live trading.

Resources
Kite Connect API Documentation
Kite Connect Python Client
TA-Lib Documentation
