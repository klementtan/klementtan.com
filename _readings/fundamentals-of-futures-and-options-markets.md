---
title: "Readings: Fundamentals of Futrues And Options Markets"
excerpt: "Notes for Fundamentals of Futrues And Options Markets"
toc: true
---

## Chapter 1: Introduction

### 1.1 Futures Contracts

Futures contract: an agreement to buy or sell an asset at a **certain time** and **certain price** in the future.
* Each contract will have a buyer who will pay and receive the asset on the delivery date and a seller who will deliver the product on the date.
* Long future position: buying the asset at a certain time and price in the future
* Short future position: selling the asset at a certain time and price in the future
* Futures price: the price that is being executed at
* Spot price: the price right now (immediate delivery)

### 1.3 Over-The-Counter Market

* OTC: trades are done over phone between buyer and seller instead of an exchange
* Market Makers: always prepared to quote both a bid price and an offer price

### 1.5 Options Contract

Options: gives the holder the right (but not obligation) to buy or sell an asset for a certain price (strike price) by a certain date (maturity)
* Call option: right to buy
    * Only makes money if the spot price is above the strike price -> buy cheaper
    * The price of the contract goes up as the strike price decreases - chances of the spot being lower than strike price (lose money) is low
* Put option: right to sell
    * Only make money if the spot price is below the strike price -> sell higher
    * The price of the contract goes up as the strike price increase - chances of the spot being higher than strike price (lose money) is low
* Types:
    * American: exercise the option any time
    * European: exercise the option only at maturity
* Option vs futures:
    * Futures require no cost to enter a contract (only pay on maturity)
    * Option will pay a premium for the contract
* Parties:
    * Buyers of calls
    * Sellers of calls
    * Buyers of puts
    * Sellers of puts 
    
### Hedgers

**Hedge Funds**: accepts from financially savvy individuals and are susceptible to lesser regulations

**Mutual Funds**: accept money from anyone and face more regulations (cannot short sell)

**Hedging using forward (OTC futures) contracts**:
* If an importer from the USA is importing goods from a company in the UK and the goods are paid in GBP
* Importer is can buy forward contracts for GBP -> buy GBP with USD at a fixed price in the future
  * The FX rate on the delivery date will not affect anything

**Hedging using Options**:
* If you hold 1000 shares of MSTF and would like to hedge your risk against the stock going down
* Purchase puts at a strike price that is slightly lower than the current price
    * This means that if the the price of the stock goes below the strike price, it will not affect you and you can still sell at the strike price
* Hedging with options will incur an option premium and will reduce the gains

**Futures vs Option**:
* Future: designed to **neutralize risk** - any price movement will not affect the outcome of it
* Options: designed to be **insurance**
    * if you are long MSFT and hold stocks, you can buy puts to hedge against the stock going down
    * but if the stock goes up you will still be able to experience the gains - only cost is the option premium

### Speculators

**Speculating with futures**:
* If the spot price is higher than the future price, you can this means you can buy the product at a cheapr price in a later date
    * If the spot is lower than you will buy the spot instead of the future
* At maturity holding spot will give you less gains than futures as the entry price of the future is lower

**Speculating with Options**:
* The option premium is much lower than the spot price -> you are able to buy much more option contracts than the underlying contract
* If the asset goes up in price a lot, the option gain is `(SPOT_PRICE - STRIKE_PRICE)*QTY - PREMIUM`
* If the price goes in the opposite direction and the option is out of the money, the option will become worthless and the spot will still be worth the low price

## Chapter 2: Mechanics of Futures Markets
