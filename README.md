
# **Trend-Aware Crypto Predictor**
## **ML + Sentiment Analysis + Autoregressive Forecasting**

A fully-functional, end-to-end crypto short-horizon forecasting system using **real market data**, **Reddit sentiment**, and **news sentiment**, powered by **LightGBM, XGBoost, and Gradient Boosting**.
The system collects live data, preprocesses it, builds features, trains models, and finally generates **7–10 minute autoregressive predicted price paths** plotted over real candlesticks.

---

## **Key Features**

* **Real OHLCV data** (Binance or CCXT)
* **Reddit sentiment extraction** (pushshift.io / Reddit API)
* **News sentiment** (NewsAPI)
* Time-aware **feature engineering**
* **Three ML models trained in parallel:**

  * LightGBM
  * XGBoost
  * GradientBoostingRegressor
* **Autoregressive next-minute forecasting**
* **Realistic noisy predictions**
* **Candlestick plotting with predicted paths**
* Full modular pipeline:

  1. `data_collector.ipynb`
  2. `preprocessor_and_train.ipynb`
  3. `main.ipynb`
* `.env` support for all API keys
* Clean folder structure, easy to extend

---

# **Project Structure**

```
|-- data/
|   |-- raw.csv                   
|
|-- models/
|   |-- lgb_model.pkl
|   |-- xgb_model.pkl
|   |-- gb_model.pkl
|   |-- scaler.pkl
|   |-- features_list.pkl
|
|-- data_collector.ipynb       
|-- preprocessor_and_train.ipynb
|-- main.ipynb       
|
|-- .env                        
|-- .gitignore
|-- README.md
```

---

# **1. Environment Setup**

### **Install dependencies**

```
pip install pandas numpy scikit-learn lightgbm xgboost mplfinance matplotlib joblib python-dotenv requests
```

---

# **2. Create `.env` (Important!)**

Create a file named `.env` in the project root:

```
REDDIT_CLIENT_ID=your_id
REDDIT_CLIENT_SECRET=your_secret
REDDIT_USER_AGENT=your_user_agent

NEWS_API_KEY=your_news_api_key
```

```python
from dotenv import load_dotenv
load_dotenv()
```

Check values:

```python
import os
print(os.getenv("REDDIT_CLIENT_ID"))
```

If it prints your key → you're good.

---

# **3. Run Notebook 1 — Data Collector**

`data_collector.ipynb` performs:

### ✔ Fetch live OHLCV (1-min)

### ✔ Fetch last 60 mins Reddit sentiment

### ✔ Fetch news sentiment

### ✔ Merge everything into a single row per minute:

* ret1
* lag returns
* rolling std
* volume change
* reddit_count_60m, reddit_sent_60m
* news_count_60m, news_sent_60m
* timestamp, symbol, OHLCV

### ✔ Save result:

```
data/raw.csv
```

---

# **4. Run Notebook 2 — Preprocessing + Training**

`preprocessor_and_train.ipynb`:

### 🔹 Builds all ML features:

* ret1
* ret_lag_1/2/3/5/10
* roll_std_10
* vol_lag_1
* reddit + news sentiment

### 🔹 Creates `target = next-minute return`

### 🔹 Splits train/test chronologically

### 🔹 Scales features with StandardScaler

### 🔹 Trains models:

* **LightGBMRegressor**
* **XGBRegressor**
* **GradientBoostingRegressor**

(Including optional regularization parameters.)

### 🔹 Saves output:

```
models/lgb_model.pkl
models/xgb_model.pkl
models/gb_model.pkl
models/scaler.pkl
models/features_list.pkl
```

---

# **5. Run Notebook 3 — Main Predictor**

`main.ipynb`:

### **Loads trained models + scaler**

### **Loads the latest row from raw.csv**

### **Simulates autoregressive next-minute predictions for 7–10 minutes**

Each model predicts:

* next minute return
* return → price update (p * (1 + r))
* fed back into next prediction
* small noise added for realism

### **Plots candlestick for the last 90 minutes + predicted paths**

You get a chart like:

```
BTC/USDT — Candles + 9-minute predicted paths
(LightGBM vs XGBoost vs GradBoost)
```

With three separate prediction trajectories.

---

# **6. Example Prediction Table**

```
Timestamp           LGB        XGB        GB
2025-01-01 12:01   10123.54  10112.83  10110.25
2025-01-01 12:02   10124.12  10111.92  10109.73
...
```

---

# **7. How to Use the Application**

### **Step 1 — Collect data**

Open:

```
data_collector.ipynb
```

Run all cells → generates `data/raw.csv`.

---

### **Step 2 — Train models**

Open:

```
preprocessor_and_train.ipynb
```

Run all cells → saves models in `/models`.

---

### **Step 3 — Predict**

Open:

```
main.ipynb
```

It will:

* load last 90 mins candles
* run all models
* simulate 9-min future paths
* plot candlestick chart with predictions
* print price forecast table

---

# **8. Tech Stack**

| Component   | Technology                          |
| ----------- | ----------------------------------- |
| Market Data | CCXT / Binance API                  |
| Reddit Data | Reddit API or Pushshift             |
| News Data   | NewsAPI                             |
| ML Models   | LightGBM, XGBoost, GradientBoost    |
| Scaling     | StandardScaler                      |
| Forecasting | Autoregressive recursive prediction |
| Plotting    | matplotlib + mplfinance             |
| Deployment  | Jupyter Notebooks                   |
| Secrets     | python-dotenv                       |

---

# **9. References**

* LightGBM documentation
* XGBoost documentation
* Sklearn GradientBoosting
* mplfinance
* Reddit API documentation
* NewsAPI.org

---

# **10. Future Improvements**

* Add LSTM or Temporal Convolution Networks
* Combine all models into an ensemble meta-predictor
* Add volatility clustering / GARCH-style features
* Replace simple autoregressive loop with a full RNN
* Add backtesting engine
* Add a live “streaming” mode
