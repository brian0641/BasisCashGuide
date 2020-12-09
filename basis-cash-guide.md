# Practical Guide to Navigating the Basis Cash Monetary Expansion/Contraction Cycles

Basis Cash is a is an open-source, permissionless algorithmic stablecoin with the goal of reviving the original [basis.io](http://basis.io/) project. The offical docs are [here](https://docs.basis.cash/). 

The stablecoin token of the Basis Cash system is the BAC token. When the protocol is mature, the value of BAC should should be relatively stable at one dollar, and can be used similarly to other popular stablecoins such as DAI, USDC, or USDT. Unlike other algorithmic stablecoin projects, the number of BAC in your wallet won't unpredictably rebase, and there is no opportunity-lost cost in failing to stake your BAC to obtain more BAC. BAC holders can use BAC to do normal stably-coin things.

The Basis Cash protocol also includes the Basis Share (BAS) and Basis Bond (BAB) tokens. This guide is for people that want to participate more deeply in the Basis Cash protocol by holding the BAS/BAB tokens. 

## Price Oracle
Decisions relating to expansion of the BAS monetary supply, as well as pricing of the BAB tokens, are controlled by a price Oracle. The current Oracle uses the Uniswap BAC-DAI pool and is updated no more than once every 24 hours as the Uniswap 24 hour TWAP. It is effectively an average of the BAC-DAI price over last 24 hours. 

The main advantage of using the 24 hour TWAP as the Oracle price is its resistance to manipulation. The disadvantage is lag. It is possible that the current spot-price of BAC could be significantly different than the current Oracle price.

## Monetary Expansion

New BAC is minted by calling the `allocateSeigniorage()` in the Treasury contract, and can only be called once every 24 hours. The first call to `allocateSeigniorage()` will not occur until Dec 11 (Fri) 12:00am UTC. There is countdown timer here: [timer](https://app.basis.cash/boardroom). 

`allocateSeigniorage()` will only mint new BAC (seigniorage) when the Oracle price is greater than 1.05. The amount of seigniorage is calulated as: `current_bac_total_supply * (oracle_price - 1)`, where `current_bac_total_supply` is the total BAC supply minus the quantity of BAC currently reserved for future BAB purchases (the Treasury reserve). For the first `allocateSeigniorage()` call on Dec 11 (Fri) 12:00am UTC, `current_bac_total_supply` is 50000.  

You can get the current BAC total supply from the `totalSupply()` function of the BAC contract and the current Treasury reserves from the `getReserve()` function of the Treasury contract. Remember to divide by 1e18 to get "normal" units.

For each call to `allocateSeigniorage()`, all the seigniorage will either be assigned to the Boardroom or to the Treasury Reserve. Specifically, if the current Treasury reserve amount is greater than 1000, the seigniorage will go to the Boardroom, if less than or equal to 1000, it will go to the Treasury Reserve. The Treasury reserve is initially set at 1001, which means the first seigniorage will go to the Boardroom.

### Boardroom

The Boardroom is the contract used to distribute BAC seigniorage to BAS holders. The rules for dividing up the BAC seigniorage are simple: seigniorage assigned to the Boardroom is given out pro-rata to all BAS shares that are staked in the Boardroom at the time of the seigniorage distribution. As an example, if 100 BAC are assigned to the Boardroom via seigniorage and you own 10% of all the BAS staked at at the seigniorage transaction, you are assigned 10 of the 100 BAC. BAC can be withdrawn from the Boardroom as soon as the seigniorage is minted to the Boardroom.

The dApp to interact with the Boardroom is at (https://app.basis.cash/boardroom). If you want to interact directly with the Boardroom contract, `stake(uint BAS_amount)` is used to stake your BAS, `claimDividends()` claims your BAC, `withdraw(uint BAS_amount)` unstakes BAS and claims BAC, and `exit()` unstakes all your BAS and claims your BAC.

**Important:** Due to a small glitch in the deployment of the contracts, to claim a portion of the first seigniorage, you will need to stake to the "old" Boardroom contract. For subsequent seigniorage events, you will need to unstake from the "old" Boardroom contract and re-stake to the new one. This will be reflected in the UI.
    

## Bonds (BAB)

Basis Bonds (BAB) are the mechanism used to remove BAC from circulation during period of contraction (i.e., when the Oracle price is below 1.0). When the Oracle price is below 1.0, users can spend BAC to buy BAB. The BAC is burnt and BAB is issued to the user. BAB is issued at a discount, which increases as the Oracle price goes lower. BAB can only be purchased when the Oracle price is less than one. Due to Oracle lag, it is possible that the spot price of BAC can be trading at well below one, but yet no BAB can be purchased until the next Oracle update. 

The price of one BAB, in DAI, is: `(oracle_price)^2`.

Since BAC is burnt to generate BAB, it might be easier to think of the BAB price in terms of BAC. For each BAC burnt, the amount of BAB received is: `1/oracle_price`.

If BAC is trading below one, and you believe that the protocol will work and BAC will eventually get back to parity, you have two options as a trader:

**Buy and Hold the Discounted BAC**: This is less aggressive than buying BAB. Your potential return is not as great as buying BAB, but the advantage is you can sell the BAC at any time.

**Buy BAB**: If you buy BAB you can potentially get bonus pricing due to the quadratic BAB pricing formula. The disadvantage is that you cannot redeem your BAB for BAC until the Oracle price is above 1.05 and the Treasury reserves have suffient BAC. When the Treasury does get sufficient reserves, BAB redeeming is done on a fist-come-first-serve basis. In other words, if there is a lot of issued BAB, it may take a number of days of seigniorage until enough BAC is minted to redeem all the BAB.

There is no advantage to buying BAB early in an Oracle period. You will get the same price (in terms of BAB issued for BAC burnt) if you wait until just before the end of Oracle period. Of course you may want to buy BAC on the spot market early in the Oracle period. 




     
