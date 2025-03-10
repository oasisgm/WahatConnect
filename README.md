import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import yfinance as yf
import os
import logging
from datetime import datetime, timedelta
from dotenv import load_dotenv
import talib
import openai
from kiteconnect import KiteConnect
import speech_recognition as sr
import joblib
from sklearn.model_selection import ParameterGrid
import time

# Load environment variables
load_dotenv()

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Securely fetch API keys from environment variables
API_KEY = os.getenv("KITE_API_KEY")
API_SECRET = os.getenv("KITE_API_SECRET")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
REQUEST_TOKEN = os.getenv("KITE_REQUEST_TOKEN")  # Fetch from env
openai.api_key = OPENAI_API_KEY

# Initialize KiteConnect
kite = KiteConnect(api_key=API_KEY)

# Function to fetch and set access token (using request token from env)
def fetch_and_set_access_token(api_key, api_secret, request_token):
    """Fetches a new access token and sets it for the KiteConnect instance."""
    logging.info("Fetching new access token.")
    try:
        data = kite.generate_session(request_token, api_secret=api_secret)
        access_token = data["access_token"]
        kite.set_access_token(access_token)
        logging.info("Access token fetched and set successfully.")
    except Exception as e:
        logging.error(f"Error fetching access token: {e}")
        raise

# Attempt to set access token
if REQUEST_TOKEN:
    fetch_and_set_access_token(API_KEY, API_SECRET, REQUEST_TOKEN)
else:
    logging.error("KITE_REQUEST_TOKEN not found in environment variables.")

# Function to fetch full historical data
def get_full_stock_history(symbol, interval="day"):
    """Fetches all available historical stock data since the company was listed."""
    logging.info(f"Fetching historical data for {symbol} with interval {interval}.")
    try:
        instrument = kite.ltp(f"NSE:{symbol}")
        instrument_token = instrument[f"NSE:{symbol}"]["instrument_token"]
    except Exception as e:
        logging.error(f"Error fetching instrument token for {symbol}: {e}")
        return pd.DataFrame()

    full_data = pd.DataFrame()
    end_date = datetime.now()
    start_date = end_date - timedelta(days=3650)

    if interval == "minute":
        delta = timedelta(days=30)
    elif interval == "hour":
        delta = timedelta(days=365)
    else:
        delta = timedelta(days=2000)

    while start_date < end_date:
        next_date = start_date + delta
        if next_date > end_date:
            next_date = end_date

        try:
            data = kite.historical_data(instrument_token, start_date, next_date, interval)
            df = pd.DataFrame(data)
            full_data = pd.concat([full_data, df], ignore_index=True)
        except Exception as e:
            logging.error(f"Error fetching data from {start_date} to {next_date}: {e}")
            break

        start_date = next_date

    logging.info(f"Fetched {len(full_data)} records for {symbol}.")
    return full_data

# Function to apply technical indicators
def apply_technical_indicators(df, rsi_period=14, macd_short=12, macd_long=26, sma_period=50):
    """Calculates RSI, MACD, and SMA for stock analysis."""
    logging.info("Applying technical indicators.")
    try:
        df["RSI"] = talib.RSI(df["close"], timeperiod=rsi_period)
        df["MACD"], df["MACD_signal"], _ = talib.MACD(df["close"], fastperiod=macd_short, slowperiod=macd_long)
        df["SMA"] = talib.SMA(df["close"], timeperiod=sma_period)
    except Exception as e:
        logging.error(f"Error applying technical indicators: {e}")
    return df

# Function for AI-based trade recommendation
def ai_trade_recommendation(df):
    """Provides trade recommendations with risk analysis."""
    logging.info("Generating AI-based trade recommendation.")
    if df.empty:
        logging.warning("DataFrame is empty. Cannot generate recommendation.")
        return "No data available for analysis."

    latest = df.iloc[-1]
    risk_reward_ratio = 2
    stop_loss = latest["close"] * 0.97
    take_profit = latest["close"] * 1.06

    if latest["RSI"] < 30 and latest["MACD"] > latest["MACD_signal"]:
        return f"🔹 Strong Buy Signal | SL: ₹{stop_loss:.2f} | TP: ₹{take_profit:.2f} | Risk-Reward: {risk_reward_ratio}"
    elif latest["RSI"] > 70 and latest["MACD"] < latest["MACD_signal"]:
        return f"🔻 Strong Sell Signal | SL: ₹{stop_loss:.2f} | TP: ₹{take_profit:.2f} | Risk-Reward: {risk_reward_ratio}"
    else:
        return "⚠ No clear trend. Hold or wait for confirmation."

# Function to execute trade
def execute_trade(symbol, quantity, order_type, stop_loss=None, take_profit=None):
    """Executes a trade with optional SL/TP."""
    logging.info(f"Placing {order_type} order for {quantity} shares of {symbol}.")
    try:
        order_id = kite.place_order(
            variety=kite.VARIETY_REGULAR,
            exchange=kite.EXCHANGE_NSE,
            tradingsymbol=symbol,
            transaction_type=order_type,
            quantity=quantity,
            order_type=kite.ORDER_TYPE_MARKET,
            product=kite.PRODUCT_MIS
        )
        logging.info(f"Trade Executed! Order ID: {order_id}")
        if stop_loss and take_profit:
            logging.info(f"SL: ₹{stop_loss:.2f}, TP: ₹{take_profit:.2f}")
        return order_id
    except Exception as e:
        logging.error(f"Error executing trade: {e}")
        return None

# Function for voice-based trading command
def voice_command_trading():
    """Allows users to place trades via voice commands."""
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        logging.info("🎙 Listening for trade command...")
        audio = recognizer.listen(source)

    try:
        command = recognizer.recognize_google(audio).lower()
        logging.info(f"Recognized command: {command}")
        if "buy" in command:
            execute_trade("TCS", 1, kite.TRANSACTION_TYPE_BUY)
        elif "sell" in command:
            execute_trade("TCS", 1, kite.TRANSACTION_TYPE_SELL)
        else:
            logging.info("Invalid command")
    except Exception as e:
        logging.error(f"Error recognizing voice command: {e}")

# Backtesting function
def backtest(symbol, params):
    """Backtests the strategy with given parameters."""
    df = get_full_stock_history(symbol)
    if df.empty:
        return 0

    df = apply_technical_indicators(df, **params)
    df["signal"] = 0
    df.loc[(df["RSI"] < 30) & (df["MACD"] > df["MACD_signal"]), "signal"] = 1
    df.loc[(df["RSI"] > 70) & (df["MACD"] < df["MACD_signal"]), "signal"] = -1

    df["returns"] = df["close"].pct_change().shift(-1) * df["signal"]
    return df["returns"].cumsum().iloc[-1]

# Parameter optimization function
def optimize_parameters(symbol):
    """Optimizes technical indicator parameters."""
    param_grid = {
        "rsi_period": range(10, 20),
        "macd_short": range(10, 15),
        "macd_long
