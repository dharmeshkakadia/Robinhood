# Robinhood_Trader

This fork adds the functionality to use this as a package/library you can install with pip/poetry.

To use with poetry, run:

``
poetry add git+https://github.com/dharmeshkakadia/Robinhood
``

To use with poetry, run:

``
pip install git+https://github.com/dharmeshkakadia/Robinhood
``

-------------------
This project was originally a fork of Jamonk's Unoffical Robinhood Api. 
However this project is considered to be a seperate project
as it contains significant changes. 

Changelist:  
 + Implemented Quote/Order objects for user-convienance 
 + Added certain Trader features to the Order object for convience (update, canceling, querying state) 
 + Fixed/Added support for quoting crypto currencies
 + Fixed/Added support for buying/selling crypto currencies
 + Removed/Changed buy/sell interface 
 + Removed all quote-utility methods
 + Removed all usages of prompts (except for login)
 
------------------

### Getting Stared:
```python
from robinhood import Trader
trader = Trader('username', 'password') 
```
Logging in will prompt for an access_code which may be submitted  via a console prompt.   
Note: You must have 2-factor authentication requried on your robinhood account. 
```python
trader.save_session('filename')
```
Saves your current robinhood session for skipping logging in (until the cookie expires).  
 To restore your session:
```python
trader = Trader.load_session('filename')
```
### Trading (Small example) 
```python 
from robinhood import Trader
trader = Trader.load_session('filename')
order = trader.buy('aapl', quantity=1)

if not order.filled():
  order.cancel()

if order.canceled():
  quote = trader.quote('aapl')
  if quote.ask < 250.0: 
    order = trader.buy('aapl', quantity=1, price=250.0) # limit order
  else:
    order = trader.buy('aapl', quantity=1, price=260.0, stop_price=255.0)  # stop-limit order 
trader.sell('aapl', quantity=1, trailing_stop_amount=5)


# crypto trading:
# buying and selling crypto is accessed via the 'crypto' property of the trader object
# (This was added as there is both a "neo" stock and a "neo" crypto currency)

dollar_amount = 500
crypto_order = trader.crypto.buy('btc', dollar_amount)              # buy 500$ worth of bitcoin (market order)
crypto_order = trader.crypto.sell('btc', dollar_amount, price=6850) # sell 500$ worth of bitcoin at 6850 (limit order)
crypto_order = trader.crypto.buy('btc', quantity=1, price=6850)     # sell 1 bitcoin at 6850 (limit order)

# optionally you may do something like for the sake of convienance: 
trader = trader.crypto 
trader.buy('btc', dollar_amount)  
trader.sell('btc', dollar_amount, price=6850)
trader.buy('btc', quantity=1, price=6850)
```

### Trader methods 

#### Logging in and Sessions
```python
 - login(username: str = None, password: str = None)  # prompts for input if username and password are not supplied.
 - logout()
 - save_session(session_name: str)
 - load_session(session_name: str) @staticmethod 
```
#### Stock Data
```python
 - instrument(symbol: str)
 - quote (symbol: str)
 - fundamentals(symbol: str)
 - orderbook(symbol: str)        # requires robinhood gold
 - watch_orderbook(symbol: str)  # actively watch the orderbook (pretty format)
 - historical quotes(symbol: str)
```
##### Crypto Stock Data
```python
 - quote (symbol: str)
```

#### Account Data 
```python
 - account()
 - orders()                         # returns order history 
 - order(order:Order)               # returns an updated order object from an existing Order 
 - portfolios()                     # not supported for 'crypto trader'
 - dividends()                      # not supported for 'crypto trader' 
 ```
##### Crypto Account Data
```python
 - account()
 - orders()                        
 - order(order:CryptoOrder)
```

#### Trading 
```python
 - buy(  
       symbol: str,                   # the stock symbol
       quantity: number,              # number of shares
       price: float = None,           # limit order if specified
       stop_price: float = None,      # stop order if specified
       trailing_stop_percent = None,  # trailing stop percent order (int) 5 -> trailing stop of 5%) 
       trailing_stop_amount = None,   # trailing stop price order 
       time_in_force = None,          # defaults to gfd (good for day)
       extended_hours = None)         # defaults to False if not suppplied 

 - sell(  
       symbol: str,                  # the stock symbol
       quantity: number,             # number of shares
       price: float = None,          # limit order if specified
       stop_price: float = None,     # stop order if specified
       trailing_stop_percent = None, # trailing stop percent order (int) 5 -> trailing stop of 5%) 
       trailing_stop_amount = None,  # trailing stop price order 
       time_in_force = None,         # defaults to gfd (good for day)
       extended_hours = None)        # defaults to False if not suppplied 
```
##### Crypto Trading
```python
 - buy(  
       symbol: str,                   # the stock symbol
       price_quantity: float = None,  # buy a quantity of crypto of equal value to the given price_quantity,
       quantity: number = None,       # number of shares
       price: float = None,           # limit order if specified
       time_in_force = None)          # defaults to gtc (good till canceled)

 - sell(  
       symbol: str,                   # the stock symbol
       price_quantity: float = None,  # sell a quantity of crypto of equal value to the given price_quantity,
       quantity: number,              # number of shares
       price: float = None,           # limit order if specified
       time_in_force = None)          # defaults to gtc (good till canceled)
       
 - cancel(order: Order/CryptoOrder)   # cancels an existing order, returns response object, success does not ensure the order has been canceled). (Robinhood response does not indicate if the order was successfully canceled) 
 ```
 - for crypto buy/sell: 'price_quantity' is mutually exclusive with quantity. 
 - trailing_stop_percent, trailing_stop_amount, and stop_price are mutually exclusive arguments. 
 - supplying `price` and `stop` argument will create a `stop-limit` order. 
 - `trailing_stop_percent`, and `trailing_stop_amount` are not compatible with `price` (RH does not support trailing-limit orders) 
 - USE TRAILING_STOPS WITH CAUTION, RH has recently been changing their implementation of trailing-stops which has periodically broken this API. PLEASE ENSURE YOUR ORDERS ARE EXECUTING BEFORE USAGE. (I have seen trailing-stop orders being submitted but will get stuck in "pending"). (Currently these stops are working, but I do not know when RH will update their API).   
 - For crypto-currencies, decimal quantities are supported. 

### The Quotes 

 - The quote object wraps a robinhood quote json and supplies convenience functionality to it. 
 - to access the underlying json use `._dict`
 - Each property will on-the-fly convert to the appropriate type, 
   this ensures that access to the original value is always available, (in case float conversion causes a loss of precision) 

#### Regular Quote 
##### Properties
```python
 - ask                     -> float
 - bid                     -> float
 - mark                    -> float   # market_price or last_trade_price (regular stocks json contains a "last_trade_price", crypto json contains a "mark_price" 
 - previous_close          -> float
 - adjusted_previous_close -> float
 - ask_size                -> int
 - bid_size                -> int
```
#### The CryptoQuote Object 
##### Properties
```python
 - ask  -> float
 - bid  -> float
 - mark -> float
 - high -> float
 - low  -> float
 - open -> float 
```
### The Orders 
 - The order objects wrap the order json and supply convenience definitions for basic functionality 
 - Properties are converted to their apropriate type on the fly, use `_dict`, to access the underlying json. 
 - Note, there are slight differences between crypto/regular orders. 
 - Orders created via Trader methods `orders` and `crypto_orders` will not have the 'time' property. 

##### Methods 
```python
 - update()                              # updates the internal `_dict` by making a request to RH 
 - cancel()                              # attempts to cancel the order,success does not indicate successful cancelation
 - filled  (update: bool = True) -> bool # returns if the order has been filled, if update is true, will call update prior.
 - canceled(update: bool = True) -> bool # returns if the order has been canceled, if update is true, will call update prior.
 - status  (update: bool = True) -> str  # returns the current status of the order
 - is_open (update: bool = True) -> bool # returns True if the order is not canceled or filled
```
##### Properties 
```python
 - time     -> pd.Timestamp   # returns timestamp when we received the response from robinhood (not RH's timestamp!)
 - price    -> float          # price  
 - side     -> str            # 'buy' or 'sell'
 - quantity -> (int if regular order, float if crypto_order)
```

---------------------
#### Original fork: https://github.com/robinhood-unofficial/Robinhood
