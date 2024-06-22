

## Objective

Design and implement a front-end for options strategy risk and reward analysis using Vue.

## Brief

Your challenge is to create a Vue component that can generate a risk & reward graph for options strategies. The component should accept an input of up to four options contracts and output the following:
1. A risk & reward graph where X is the price of the underlying at the time of expiry and Y is the profit/loss at that price. 
2. Max profit, max loss, and all break even points.

### Evaluation Criteria

- Completeness of the logic
- Usability of the graph
- Aesthetics of the UI
- Readability and code structure

### CodeSubmit 

Please organize, design, test, and document your code as if it were
going into production - then push your changes to the main branch.

Reply to the invitation e-mail with your github username to notify of completion.

Have fun coding! 🚀
The Aries Financial Team


# Notes and research

### Financial Wisdom

https://youtu.be/XV9avMhNL2Y?si=zUJw9YktRTDAOumR

Exercise/strike price is the pre-determined cost of shares.

Premium is the predetermined cost of the contract for the buyer.

Option buyer has the right but not the obligation to honor the contract.

Option seller is obligated to comply if buyer chooses to exercise his right.

Call buying; gives buyer the right to buy securities at a predetermined price. 
 - bull buys because he expects the price to rise
 - bear sells for the premium and/or he wants out of the security anyways

Put buying; gives buyer the right to sell securities at predetermined price.
 - bear buys because he expects the stock price to fall
 - bull sells for the premium and/or he wants the stock anyways.

Pricing premiums
 - Intrinsic value; the total value?
 - Time value; how close to expiration
 - Implied volatility; higher volatility, higher cost

Option buyers
 - Unlimited upside
 - Max loss = Premium paid

Option Sellers
 - Unlimited downside
 - Maximum profit = Premium recieved

How to profit

track/estimate trends with;
 - quantitative trends; historical prices
 - qualitative trends; sentiment

options for; stocks, index, commodities or currencies

don't risk more than 2% of capital on any trade

### Wysetrade

https://www.youtube.com/watch?v=wQ2PXJgcsxk

In/at/out the/of money; whether the option is profitable

the greeks
 - Delta(d); sensitivity to price changes of the underlying asset
 - Gamma(Y); delta sensitivity to price changes of the underlying asset
 - Theta(0); sensitivity to passage of time, aka time decay
 - Vega(v); sensitivity to changes in volatility
 - Rho(p); sensitivity to interest rate changes

Long Call; buy call when you expect a security to rise
 - profit = current price - strike price - premium

Long Puts; buy put when expect a security to fall
 - profit = strike price - current price - premium

Covered Calls; When an investor owns a stock and sells a call option for it
 - protect owner in the case of a decline in value
 - generate a stream of income from securities you're holding anyways
 - trading possible higher returns above strike price for cash now
 - good option when you want to hold onto a stock but you expect the price to stagnate over the duration of the option
 - you may be forced to sell the stock if its value goes above the strike price

Protective Puts; When an investor owns a stock and buys a put option for it
 - limits your potential loss to the difference between current price and strike price + premium
 - you always lose the premium
 - insurance policy for your stock's earnings
 - predetermined exit strategy that protects your initial investment

### Wallstreet Mojo

Example profit chart;

https://www.wallstreetmojo.com/option-contract/

### The Balance: How Options are Traded

https://www.thebalancemoney.com/how-options-are-traded-1031302

 Long Trade

    Entry type: Buy a Call or a Put
    Profit potential: Unlimited (call) or underlying strike price value minus premium paid (put)
    Risk potential: Limited to the options premium

Short Trade

    Entry type: Sell a Call or a Put
    Profit potential: Limited to the options premium
    Risk potential: Unlimited

### Everything You Need To Know About Options Bid Ask Spread

https://optionstradingiq.com/options-bid-ask-spread/

The bid price is the best (highest) price someone is willing to buy the instrument for.

### Investopedia

https://www.investopedia.com/ask/answers/100314/whats-difference-between-long-and-short-position-market.asp

 - With stocks, a long position means an investor has bought and owns shares of stock.

 - An investor with a short position has sold shares but does not possess them yet.

 - With options, buying or holding a call or put option is a long position; the investor owns the right to buy from or sell to the writing investor at a certain price.

 - Conversely, selling (or writing) a call or put option is a short position; the writer must sell to or buy from the long position holder or buyer of the option.

The ask price is the best (lowest) price someone is willing to sell the instrument for.

Bid = buy. Therefore, the buyer wants the lowest possible price.

Ask = sell. Therefore, the seller wants the highest possible price.

The mid prices is therefore right in between where the buyers and sellers are.

### How to read an option chain

https://www.businessinsider.com/personal-finance/option-chain?op=1


### Charles Schwab: Options Exercise, Assignment, and More: A Beginner's Guide 

https://www.schwab.com/learn/story/options-exercise-assignment-and-more-beginners-guide

 - A long call exercise results in buying the underlying stock at the strike price.
 - A short call assignment results in selling the underlying stock at the strike price.
 - A long put exercise results in selling the underlying stock at the strike price.
 - A short put assignment results in buying the underlying stock at the strike price.

