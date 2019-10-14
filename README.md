# A Python Implementation of a Limit Order Book

A limit orders describes:

  1. If it is a buy (bid) or a sell (ask) order
  2. The maximum quantity to trade
  3. The limit price. For a buyer this is the maximum they will pay for a share (or contract). For a seller, it is the opposite - It is the least they are willing to accept.

A trade needs both a buyer and a seller. The exchanges role is to get the buyers and seller the best possibile price, through a process is called 'matching'. The matching rules determines which orders get filled and which stay within the Limit Order Book (LOB).

The algorithm used in this application is called the Price Time algorithm. This is the most common algorithm used in exchanges today, but there are other rules worth of note - The Price ProRata and the Price TimeProRata rules.

Not everything trades this way. Some brokers fill orders within their own brokerages, where a buyer and seller are clients with the broker, thus an exchange is not needed. Also, some brokers pay to use Dark Pools, but these topics are out of the scope of this introduction.

Consider the following orders:

| Buy Orders    	| Sell Orders   	|
|---------------	|---------------	|
| B1 : 10 @ 99  	| S1 : 30 @ 99  	|
| B2 : 40 @ 99  	| S2 : 20 @ 101 	|
| B3 : 20 @ 100 	| S3 : 10 @ 100 	|

They arrive in the following order:

![Trade Order](https://latex.codecogs.com/png.latex?\large&space;b_{1}&space;\rightarrow&space;b_{2}&space;\rightarrow&space;s_{1}&space;\rightarrow&space;b_{3}&space;\rightarrow&space;s_{2}&space;\rightarrow&space;s_{3})

```python
from ordrbook import OrderBook


order_book = OrderBook()


# B1: BUY 10 LIMIT 99
order_book.bid_limit_order({'order_id': 'b1', 'trade_id': 'b1', 'timestamp': 1,
                            'type': 'bid', 'quantity': 10, 'price': 99., 'type': 'limit'})

# B2: BUY 40 LIMIT 99
order_book.bid_limit_order({'order_id': 'b2', 'trade_id': 'b2', 'timestamp': 2,
                            'type': 'bid', 'quantity': 40, 'price': 99., 'type': 'limit'})

# S1: SELL 30 LIMIT 98 (Match! B1 Gets Priority)
order_book.ask_limit_order({'order_id': 's1', 'trade_id': 's1', 'timestamp': 3,
                            'type': 'ask', 'quantity': 30, 'price': 98., 'type': 'limit'})

# B3: BUY 20 LIMIT 100
order_book.bid_limit_order({'order_id': 'b3', 'trade_id': 'b3', 'timestamp': 4,
                            'type': 'bid', 'quantity': 20, 'price': 100., 'type': 'limit'})

# S2: SELL 20 LIMIT 101 (Price cannot be fulfilled yet. Placed on order book.)
order_book.ask_limit_order({'order_id': 's2', 'trade_id': 's2', 'timestamp': 5,
                            'type': 'ask', 'quantity': 20, 'price': 101., 'type': 'limit'})

# S3: SELL 10 LIMIT 99 (Match! B2 is part-filled.)
order_book.ask_limit_order({'order_id': 's3', 'trade_id': 's3', 'timestamp': 6,
                            'type': 'bid', 'quantity': 10, 'price': 99., 'type': 'limit'})

# See trades
list(order_book.tape)
```


There are two important points to mention, regarding broadcasting the order book publicly:

  1. Neither counterparty can know whom they were matched with. Only the exchange can know this information.
  2. Only the depth can be visible to the public. The individual trade sizes must not be made public.

You can view the order book and/or trades by using the following code:

```python
print('Trades:')

for entry in order_book.tape:
    print(f"`{entry['party1'][0]}` sold {entry['quantity']} @ £{entry['price']} to `{entry['party2'][0]}`")

print('Order Book:')
print('Bids:')

if order_book.bids != None and len(order_book.bids) > 0:
    for key, value in order_book.bids.tree.items(reverse=True):
        print(f'Price: {key}, Volume: {value.volume}')

print('Asks:')

if order_book.asks != None and len(order_book.bids) > 0:
    for key, value in order_book.asks.tree.items():
        print(f'Price: {key}, Volume: {value.volume}')
```

----

### Installation

`pip install ordrbook`
