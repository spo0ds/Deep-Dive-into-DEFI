**Tether(USDT)**

The idea proposed by JR Willet was materialized into `mastercoin` which aimed to utilize a new `second layer` on the bitcoin blockchain. Mastercoin would eventually become the technological foundation of the Tether cryptocurrency. Then, in 2014, a few team members from the Mastercoin project established their own startup called `Realcoin` using the same second layer technology, now called `Omni Layer Protocol`. Later that year, realcoin was renamed `Tether`, refinancing its ties to fiat currency.

Each tether is equivalent to one US dollar, essentially making it a digital dollar. Bitfinex, which is a sister company of Tether, claims "every tether is always 100% backed by our reserves, which include traditional currency and cash equivalents, and from time to time may include other assets and receivables from loans made by Tether to third parties, which may include affiliated entities." Every tether is also pegged one to one to the dollar.So 1 USDT is always valued by Tether at 1 USD.

Besides being a stablecoin, it exists on several different blockchains. There's a tether token on the omni bitcoin platform, Ethereum, EOS, and Tron. The difference between the four coins has to do with the properties inherited by the blockchain hosting tether. For instance, for maximum safety and immutability, some investors may prefer the Bitcoin Omni version of tether. Other investors might prefer the ERC20 Tether token, as it can be used in DEFI to earn a passive income. Few investors appear to be interested in keeping their tethers connected to the TRON network.

How does tether work?

Tether's omni version consists of a 3-layer stack: the Bitcoin blockchain, the omni layer protocol, and Tether Limited. The Bitcoin blockchain is the bottom layer that serves as a transactional ledger and runs the consensus algorithm. The second layer, omni, is used to create and destroy digital Tether coins as well as track and report the tokens in circulation. This layer also enables users to send and store these tokens securely and anonymously. The third layer, Tether Limited, is the business entity that manages fiat deposits and withdrawals from the Tether reserve, along with the management of Tether's web wallet and compliance logistics.

When a user deposits fiat currency into Tether Limited's reserve, by selling fiat to buy USDT, the business generates and issues the corresponding amount of digital dollar tokens to the user that can be sent, stored, or exchanged. If a user deposits $100 USD, they will receive 100 tether tokens. These tether coins are destroyed and removed from circulation when the user redeems the tokens for fiat currency. Bitcoin's secure blockchain and its network effect helped Tether gain popularity over other stablecoins operating on different blockchains.

In 2019, the ERC-20 version surpassed the Omni version in number of transactions, and in October of the same year, it surpassed it in market cap. Today, the ERC-20 version is the indisputable leader in the market cap between the two. Critics claim the high volume of USDT transactions saturates the Ethereum network and increases transaction fees. In fact, when most exchanges made the switch from Omni to ERC20, the oversaturation of the trades was 17 times higher than it was during the Ethereum gridlock of November 2017.

USDT has two more versions, built on top of two more blockchains that are smart-contract oriented: TRON, EOS, and the Algorand network.

# MakerDAO

**DAI and MakerDAO**

The idea of DAI is kind of an incentive structure. Maker Foundation created the Maker DAO which is a decentralized organization. They try to create the algorithm (incentive structure) to make the users undertake certain behaviour and all those behaviours would create the token DAI that would be worth roughly a dollar. Consider how difficult it is to take all of the cryptocurrencies that are trading on centralized and decentralized exchanges, with all of their wild volatility, and try to compress all of that volatility into something worth a dollar.They are going to spit so many that whole DeFi can work using the DAI token that is worth dollar.

So they created the algorithm and gave people the incentive  for people locking the ETH in their `vaults`. Back then they were called `CDPs` and now they're called "vaults." Why would you do that? because DAI is worth roughly a dollar. They can transact on that. If I want to send $100 to someone who's around the world, I don't want to use the banking system because it's going to cost me a lot to send the money. So I might send 100 DAI.

There needs to be some collateralization ratio. Let's say that the ETH is worth $300 and I lock up 1 ETH. In essence there's 150% collateralization ratio. So ETH is the collateral. For 1 ETH worth $300, I get 200 DAI. Why should I lock my ETH instead of selling it for $300? The reason partially is because ETH is going to go up to $500. So I want to hold on to my ETH, but I want to borrow and transact in DAI. I'm borrowing DAI and locking up my ETH. They have done this algorithmically to the point where the interest that I'm going to pay is such that I think ETH is going to go up more in value than the interest I'm going to pay. If I want to unlock that ETH, I need to pay the DAI back plus some interest.


The issue we have to address is volatility. Well the 150% collateralization rate helps with that volatility because if I put in 1 ETH which is worth $300 and I only got 200 DAI back. So even if ETH goes to $275, my  health factor ir still good because they only issued 200 DAI. If the collateralization ratio is 140% and ETH goes down to $280, my position will be liquidated. The ETH was locked in a smartcontract, the vault would essentially take it and burned. My collateral would be taken, and that's how they protected the value of the DAI. The interest rate of the DAI is always changing to give people the incentive to put more into the vault or potentially take out of the vault.

The incentive structure is there to make people drop ETH in the vaults, lock it up, pull DAI out and pay some level of interest. The maker token trades kind of irrespective of this because you have to have the maker token in order to pay the interest. There's enough value in the `MKR` token to pay the interest. If I get liquidated, there's also a liquidity fee which is a fee that I pay for transaction because they have to liquidate my ETH which is no different than getting liquidated in traditional finance loan. So most people would do 200% to 300% collateralization rate because they don't want to get liquidated.

Now they also have `multi-collateral DAI`.They've enabled other cryptocurrencies to be able to use inside the vaults.This is great because we're not 100% beholden to the price of ETH.The price of ETH can go up and down but I can lock up other tokens  and produce DAI.Oracles are being used to define the price of the token.This makes ecosystem stable ideally because at some point all the different cryptocurrencies  will not move in concert together.Sometimes ETH will go up and others will go down.They move in different ways that way I might not have to worry about my collateralization or liquidation.

On Black Thursday, the entire crypto market fell and all the positions were liquidated. The value of DAI kind of went crazy for a while because of all these liquidations and it happened so quickly that the price of ETH just fell by 40%/50% in a matter of hours. But there were some governance changes that were made that got the value of DAI back to around $1.

DAI is roughly pegged to dollar. It's done algorithmatically using incentive to drive certain behaviours to lock up my crypto and borrow DAI. It mint's DAI as I'm borrowing and have to pay it back with interest. It can always be monitoring the supply and demand. If the demand curve has moved a little bit, we need to move the supply curve by changing the interest rate and get more people to actually lock up their crypto and create more DAI or get people to pay the DAI back and we burn it and take their crypto out because the interest rate is such that they don't want to lock their crypto anymore.

**Mechanism Design of Maker (DAI and MKR)**

**Dual Token Mechanism**

There are two token in the sytem which is DAI soft-pegged to US dollar and there's a secondary token MKR.This secondary token primary goal is for it to absorb volatility in the system itself.So think of it as DAI being your output which is stable(low volatility) but there's always be volatility within and outside of the system.The volatility needs to go somewhere which it goes to MKR.It is a utility token and it functions like governance token for voting, can be used to pay off interest accured in the system and during insolvency, crashes or liquidations, MKR can be minted to be sold for DAI in the ecosystem.So MKR is quite a crucial part in governing the entire system of DAI.DAI is really the facilitator to allow people to exchange for goods and services and MKR plays a governance role  in the ecosystem.Because MKR plays the governance role of this utility function, MakerDAO moved more of its funds towards DAO which you can imagine the prices of MKR soared because there's more of this utility function that the MKR token can accure.

**Reserve Mechanism**

It means that the stable coin which is backed by reserves and you can use the DAI to redeem for the uderlying collateral.It can be backed by whatever reserve but if you can't redeem  it for underlying reserve, it's not really reserve backed.Reserve backed means you're able to redeem it for the underlying.With DAI you can  redeem it for underlying.You basically redeem it for the crypto assets that you have deposited to get DAI.

**Creating DAI and How it works**

You can create DAI is three simple steps.First one is you have to own the asset.Second if you deposit the asset  into the vault and thirdly based on the amount of the value, you can mint some DAI out.The only thing that you have to be careful about is the minimum `collateralization ratio`.

**Maintaining Stability**

MKR is the one that allows them to fluctuate and that's not where they're controlled.With DAI it's fixed at $1.What kind of code we can embed into DAI so that when things fluctuate too much the code will execute and deal with it quickly? The alternative is what we have with fiat money where central banks try to manage that by interest rates via open market operations to maintain that stability and they also have long run plans trying to understand what's going on in a market and then trying to have different kind of monetary policies to deal with it.Well that is good but there's also some problems which is the monetry policy is quite laggy and market moves so much faster and sometimes the monetary policy isn't reflective of what the market conditions are.

But with programmable money  like stable coins, you can program some level of monetary policy into the code and so you can execute them according the market conditions are.What are two scenarios where DAI is not stable?First is when it's more than $1 and the second is when it's less that $1.When it's more than $1, there are two ways to deal with it.
- Short term
    - More attaractive to pedlege additional collaterals to mint more DAI(always values at $1 within the system)
        - The system always value it as $1 but the outside system when people are trading it's maybe $1.05.So if I can mint it at $1 and trade it outside at $1.05, I just earn $0.05 from nothing.It's called arbitrage.So people will take their collateral, open one of these vaults, deposit it in, borrow DAI and sell the DAI in the secondary market.This is effectively been done to increase the total supply of DAI.You have limit when secondary market is trading at $1.System can just mint DAI to increase the supply.
    - Increase DAI supply.Price falls back to $1.
- Long Term
    - Decrease the DAI savings rate
        - Small fees being generated if you deposit your DAI in the system just like a bank.Your fixed interest rate that you earn from the money that you deposit in the bank.You can earn some savings.Right now it's 0, but in the long run, you can decrease it which incentivise people from removing liquidity and then increasing the supply in the system.
    - Decrease the stability fee(interest rate)
        - When you go to the bank and you mortgage your house so bank could lend you some money and every month you'll repay some amount of money.Their amount of money is the principal and also some ineterst because the bank is giving you additional money upfront and you have to pay for the service being provided.In the same way there's the interest rate that's accured when you're depositing your ETH into the vault and borrowing DAI out.It's an interest rate but they call it `stability fee`.So when DAI is more than $1, we want to incentivise the increase of supply and so we want to decrease the stability fee so that more people  will be borrowing and increase the supply of DAI in the system.

When DAI is less than $1, the exact opposite happens.
- Short term
    - But DAI to repay debt
    - Decrease DAI supply.Price back to $1
- Long term
    - Increase DAI savings rate
    - Increase stability fee to stimulate demand

**Crashes,  Liquidations**

When we think about how stability is maintained, it's not just about how we get stability in the long run and short run of the stablecoin, but it's also important to think about what happens when the market crashes because these assets are collateralized by something else; they're backed by reserves. So when the value of the reserves fluctuates, the mechanisms to maintain that stability are quite volatile. How can we deal with that and manage that kind of volatility? There are two ways.
- Emergency shutdown to stop DAI minting
    - The whole system stops minting DAI, and you can basically start closing your vaults because you start repaying the system.
- Auction vaults to repay debts by selling collaterals in the vault.
    - You have a liquidation mechanism. People who cannot afford to pay their vault fees will have their vaults liquidated through an auction mechanism. So they will auction out the vaults, and the fees earned from the auction will be used to close and cover that vault and repay the entire system.

So having a high collateralization ratio reduces the impact that the volatility of the collateral will have on the stablecoin itself.

**How is this different from lending and borrowing?**

In the end, you're basically lending your ETH to get DAI and then using DAI in another system. It's quite similar to lending and borrowing because you're kind of like lending the bank your house, then you borrow US dollars and use them to buy groceries and live your life. Well, the difference is that the value of the asset that you're borrowing is not managed by the secondary party but by the system itself, which allows for more customization, more mechanisms, and more incentives to be programmed into the entire system. So it allows for more control over how the ecosystem can grow.

One of infrastructure's building blocks is lending and borrowing. So Maker is taking that building block and adding different kinds of building blocks, which is the creation of the currency to be added into the lending and borrowing mechanism. It functions similarly to lending and borrowing, but it is also quite different because in this system it also has a different kind of asset class, which is the currency created by the system, which means that every incentive and mechanism in place must align not only with the function of it as a lending and borrowing application, but also with the function of it as a stablecoin to maintain stability and reduce risk or volatility. That's why it's quite different.

**Transparency**

Why do we need to create stable coins on-chain when you can do it off-chain, as with Tether or USDC, which are collateralized with fiat money?

Yeah, it's possible, but is that good enough? We can take it a step further by having all assets on-chain for transparency, verifying the data on-chain, and allowing people to do risk calculations, including understanding the total amount capitalized or collateralized in the system and comparing it to other asset classes.



