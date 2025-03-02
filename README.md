
Stock Trading Automation System
📌 Overview
This project is a Stock Trading Automation System that integrates:
Zerodha KiteConnect API for placing and managing trades.
TA-Lib for technical analysis indicators (RSI, MACD, SMA).
OpenAI API for AI-based trade recommendations.
Speech Recognition for voice-based trading commands.
Pandas & NumPy for data processing.
🚀 This system fetches historical stock data, applies technical indicators, generates trade signals, and allows for automated order execution.

📦 Installation
1️⃣ Clone the Repository
git clone https://github.com/yourusername/Stock-Trading-Bot.git
cd Stock-Trading-Bot

2️⃣ Install Dependencies
pip install -r requirements.txt

3️⃣ Configure API Keys
Update the API_KEY and API_SECRET in the main script:
API_KEY = "your_api_key"
API_SECRET = "your_api_secret"

Set your OpenAI API key as an environment variable:
export OPENAI_API_KEY="your_openai_key"



🛠 Features
✅ Fetch Full Stock History
Retrieves up to 10 years of historical stock data.
📊 Apply Technical Indicators
RSI (Relative Strength Index): Detects overbought/oversold conditions.
MACD (Moving Average Convergence Divergence): Identifies trend strength.
SMA (Simple Moving Average): Smoothens price trends.
🤖 AI-Based Trade Recommendations
Buy Signal: RSI < 30 & MACD crossover.
Sell Signal: RSI > 70 & MACD crossover.
Hold Signal: No strong trend.
🎙️ Voice Command-Based Trading
Uses Google Speech Recognition to allow hands-free trading.
📈 Automated Trade Execution
Places market buy/sell orders with optional stop-loss (SL) and take-profit (TP).

🚀 Usage
Fetch Historical Stock Data
from trading_bot import get_full_stock_history
data = get_full_stock_history("TATAMOTORS")print(data.head()
Apply Technical Indicators
from trading_bot import apply_technical_indicators

data = apply_technical_indicators(data)
print(data[['RSI', 'MACD', 'SMA']].tail())

Get AI Trade Recommendation
from trading_bot import ai_trade_recommendation

signal = ai_trade_recommendation(data)
print(signal)

Execute a Trade
from trading_bot import execute_trade

execute_trade("TATAMOTORS", 10, "BUY")

Voice Command Trading
from trading_bot import voice_command_trading

voice_command_trading()

