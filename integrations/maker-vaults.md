# Maker Fixed-rate Vaults

The Deco protocol, in collaboration with MakerDAO, can offer fixed-rate vaults to Maker vault users.

The Deco user experience is simple: vault owners purchase a pure-yield token which earns a yield that offsets the entire stability fee on a vault.

A Vault need not migrate to an external protocol, either before or after the fixed rate expires, which makes it possible for the Maker protocol to retain its core business and not lose stability fees. Vault owners are not required to change any of their own vault management processes. Users may continue to use the same UI or smart contract to manage their vault. The vault itself remains fully independent, and within the Maker Protocol, permitting the vault owner to receive a fixed rate on the stability fee. Fixed rate token issuance on Deco is controlled and maintained by Maker governance, allowing the DAO to control fixed rates without relying on external protocols.

There are three main components to this integration, each of which is under the control of MakerDAO:

* A `FEE` token for each collateral type. Ex: `FEE-ETH-A`.
* A Deco instance deployed for each `FEE` token. Ex: Issues `ZERO-FEE-ETH-A` and `CLAIM-FEE-ETH-A` tokens.
* One generic `ZERO-A` collateral type setup to source Dai for `FEE` token issuance of all
collateral types. Ex: `ZERO-FEE-ETH-A`, `ZERO-FEE-WBTC-A`, et cetera, can be locked as collateral in the `ZERO-A` collateral type to generate Dai used internally for the `FEE` token issuance.

![Fixed-rate Vaults](_media/../../_media/maker-deco-diagrams.png ':size=800')

## Usage Steps

1. The vault owner purchases an additional token. Ex: 100,500 CLAIM-FEE-ETH-A tokens valid for three months to hedge the ETH-A vault with 100,500 DAI in debt for three months.
2. The purchase price of the token is the total stability fee to be paid. Ex: Vault owner pays 500 DAI to purchase these 100,500 CLAIM-FEE-ETH-A tokens, uses the 100,000 DAI in debt for their own purposes, which fixes their total vault debt at 100,500 DAI for the entire duration the CLAIM-FEE-ETH-A tokens are valid for irrespective of the changes to the ETH-A stability fee over this duration.
3. Token earns yield during its validity period which equals the actual stability fee accruing on the vault. Ex: ETH-A stability fee increases to 50% APR at some point which increases the vault debt to 113,000 DAI. 100,500 CLAIM-FEE-ETH-A tokens would earn 12,500 DAI as yield over the same duration (equal to the stability fee accrued) which will pay down debt back to 100,500 DAI.
4. Vault owner can collect yield from token as many times as they want to continuously payback accrued stability fee on their vault and eliminate any additional interim exposure to a lower collateral ratio. Ex: Vault owner can collect Dai yield from the 100,500 CLAIM-FEE-ETH-A tokens as many times as they wish to repay the additional debt being continuously added to their vault with the real stability fee.
5. Token balances are fungible and allow vault owners to hedge a portion of their debt by buying a lower number of tokens than the vault debt. Ex: Vault owner can choose to purchase only 50,000 CLAIM-FEE-ETH-A tokens to hedge a portion of their vault debt.
6. Vault owners can also sell the token on the market at any time. Ex: Vault owner can sell their 100,500 CLAIM-FEE-ETH-A tokens at any time based on the latest stability fee outlook.
7. Vault owners can also purchase a new token to extend the hedge when the current tokens expire.

## Fee Token

Fee tokens are setup to earn a yield which tracks the stability fee of a specific collateral type within the Maker protocol like ETH-A. They are like CHAI token balances, which in conjunction with the Pot contract, have a yield which tracks the Dai Savings Rate. The issuance is restricted and placed under the exclusive control of MakerDAO because the yield earned by the fee token (matching the stability fee) is a long-term liability in the financial sense.

Fee tokens are denoted with the token symbol `FEE` appended to the symbol of the collateral type. Ex: `FEE-ETH-A`. These tokens are only used for internal accounting purposes and are not allowed to circulate freely in the marketplace, or to be directly held by individual users. Fee token issuances are immediately locked within Deco to ensure that a maturity timestamp is set right at the issuance. Deco is setup to automatically burn them after the maturity timestamp which in turn removes the liability form MakerDAO to pay the yield indefinitely.

In exchange for holding this liability, MakerDAO benefits by being able to collect the entire stability fee, e.g., 3, 6, or even 12 months, up front from a vault owner versus slowly collecting a portion from the vault from each block. From a usage perspective, this is a vehicle to achieve vault user lock-in for the duration, because the entire stability fee will have been paid up-front to MakerDAO.

Deco's novel safety feature, Close, ensures that the whole outstanding FEE token supply is subject to being burned; and that all long-term obligations tied to future maturity dates do not become an impediment to the redemption process for Dai holders if an emergency shutdown is triggered.

## Deco Instance Customization

The Deco protocol integration interface is easily modified to suit the needs of the Maker protocol, MakerDAO, and vault owners utilizing the following steps:

1. MakerDAO creates a Dai balance, deposits it within a FEE contract to mint the FEE token balance.
2. Locks FEE token into Deco to issue ZERO-FEE and CLAIM-FEE token balances.
3. Locks the ZERO-FEE token in the token adapter to receive a ZERO-A internal balance within the Maker protocol.
4. ZERO-FEE vault generates debt 1:1 and uses the Dai to settle the originally generated Dai balance.
5. MakerDAO can auction the CLAIM-FEE token balance to vault owners, and collect the Dai payed as a stability Vault owner collects yield on the CLAIM-FEE token until maturity to offset the stability fee accruing on the vault, with no other impact elsewhere.
6. The CLAIM-FEE token balance is burned automatically when the yield for the entire duration is collected.
7. The ZERO-FEE token balance is redeemed for the same amount of Dai at maturity, and both ZERO-FEE and FEE token balances are burned and removed from circulation.

The base Deco protocol is easily modifiable to deploy instances that handle the needs of the Maker protocol.

### Fraction Value Capture

The Snapshot and Insert functions are customized in the following manner:

`snapshot` is a trustless function which reads the fraction value of the FEE token contract on-chain. As the stability fee of a collateral type increases, the yield of the FEE token increases correspondingly, which is reflected as a lower fraction value then recorded and stored over time within Deco.

`insert` is used to permit MakerDAO to accurately insert fraction values at maturity timestamps.

### Issuance

MakerDAO first generates Dai necessary to activate the issuance process. We will explain how the generated Dai is fully covered by a corresponding amount of debt documented at the end of the issuance process. Issuance is under the control of MakerDAO.

Only change made to `issue` is it will be placed under the control of MakerDAO.

For example, If fraction value at issuance timestamp is 0.9. MakerDAO mints 9 Mil FEE- ETH-A tokens, executes issue for a maturity timestamp. This would create 10 Mil ZERO-FEE-ETH-A and 10 Mil CLAIM-FEE-ETH-A token balances (The notional amounts track 10 Mil because 9 Mil FEE- ETH-A tokens when fraction value is 0.9 are worth/can claim 10 Mil Dai).

### Settlement

Deco typically settles its obligations in the yield token balance itself, and most integrations do not have the settlement functions modified. For this integration, Deco must perform a FEE token redemption because a FEE token cannot be distributed directly to zero and claim holders and allowed to circulate freely. All settlement functions like `redeem`, `collect`, `withdraw`, `cashZero` and `cashClaim` are setup to redeem FEE tokens for a Dai balance which is then distributed to holders. This does not break guarantees because FEE tokens are backed by MakerDAO, and can never be insolvent, and can be redeemed for Dai in any scenario.

## CLAIM-FEE Token Sales

MakerDAO can convert the CLAIM-FEE-ETH-A balance it holds at the end of the issuance process to an ERC20 balance using the balance adapter and sell them to interested vault owners through auctions using Gnosis Protocol v2.

Vault owners execute the collect function periodically (or just once after maturity) to collect yield earned in Dai and use the balance to repay any other debt accrued on the vault due to the stability fee on the collateral type.

## ZERO-A Collateral Type

New unbacked Dai is minted at the start of the issuance process, and paid immediately at the end of the issuance process by generating an equal amount of debt using the ZERO-FEE tokens which are themselves collateral within the Maker protocol.

One point to note here is that a single ZERO-A collateral type can handle the needs of a Fee token setup for all collateral types.

ZERO-A is backed 1:1 with any ZERO-FEE token. Ex: 10 Mil ZERO-FEE-ETH-A tokens would be locked as collateral to generate 10 Mil Dai, and immediately repay the exact number of Dai used at the start of the process to issue 9 Mil FEE-ETH-A tokens also worth 10 Mil Dai at a fraction value of 0.9. The collateral type need not be setup with price feeds or liquidation infrastructure.

The collateral type need not be setup with price feeds or liquidation infrastructure.

For example, after 10 Mil ZERO-FEE-ETH-A tokens mature, they can be redeemed for their notional amount, which is 10 Mil Dai, which will be able to repay the entire debt and retire the vault as well. If emergency shutdown is triggered, the same redemption will happen irrespective of the maturity timestamp and the Dai can be used as part of the redemption process.

### Token Adapter

The token adapter setup to accept an approved list of MakerDAO managed FEE tokens as deposit and issue internal gem balance for a single ZERO-A collateral type.

`join` transfers the ZERO-FEE-ETH-A token balance to the adapter and issues the ZERO-A internal gem balance to the depositor, which would be MakerDAO.

Token adapter would implement public functions that allow anyone to execute Deco redemption functions on ZERO-FEE token balances it holds. Executing `redeem` replaces the ZERO-FEE token balance the adapter holds with an equal amount of Dai.

`repay` function is implemented within the adapter to use any Dai balance it holds to repay the ZERO-A vault debt and remove both debt and Dai from circulation.

`sell` function can be implemented optionally to allow MakerDAO to trigger an auction of the ZERO-FEE tokens it holds for Dai at a discount to both incentivize long-term Dai holders as well as retire the ZERO-A vault debt early with the sale proceeds.

The Deco instance would at some point always replace the ZERO-FEE balances held by the token adapter contract with Dai. This Dai is then used to payback all debt.

`exit` is not normally used at all like in a typical token adapter, since the proceeds from all ZERO-FEE tokens deposited are normally used to payback ZERO-A vault debt. But it is setup to exit the internal ZERO-A collateral balance for Dai(the individual ZERO-FEE token balances that were original deposited are replaced by Deco) in case there is an abrupt emergency shutdown.

## Emergency Shutdown and Close

If emergency shutdown is triggered, the `calculate` function is setup to forward all Dai to Zero-FEE tokens irrespective of the maturity timestamp. All ZERO-FEE token balances held by the adapter would be immediately replaced with Dai. This allows the internal ZERO-A token balance to be converted to Dai and the collateral owed is redeemed.

To remove additional steps to redemption, Deco Fee instances can be shutdown to retire the debt in ZERO-A collateral type before the emergency shutdown in the Maker protocol is triggered.

## Summary

Fixed-rate protocols face liquidity issues; they must find buyers for both zero and claim tokens at the same time at the conclusion of the issuance process. By integrating with a collateral type, we have removed the need to find buyers for zero balances immediately. In other words, there is no need for symmetric demand, the DAO may issue claim tokens without needing to find a buyer for the zeros immediately. With the process we have engineered, the DAO can temporarily or permanently hold ZERO tokens which creates a seamless user experience for MakerDAO to satisfy the demand for CLAIM-FEE tokens without constraint.

ZERO-FEE tokens can also be unlocked from the adapter whenever it is required. The debt is setup to be automatically repaid on the maturity timestamp; or, if an emergency shutdown is triggered, or if MakerDAO wishes to shut down the Deco instance for whatever reason, all obligations will be repaid. Auctioning Zeros may also help MakerDAO discover the yield curve for Dai and can use this information to set an appropriate DSR.

Some of the benefits of this integration to vault owners and MakerDAO, include:

* Fully fixed stability fee with no upper limits: Once purchased, the token issued by Deco and MakerDAO can offset the stability fee up to any increase;
* No Vault Management Changes: Vault owners do not have to make any changes to their vault ownership or management;
* New Revenue Stream: This protocol opens an additional source of income in the form of fixed-rate risk premiums for MakerDAO and gives it the ability to upsell a new product to the largest vault owners and future RWA vaults who also tend to borrow at scale;
* Vault owner retention: Vault owner stickiness is vastly improved, especially for the largest vaults, since they have prepaid their stability fees as well as locked in a fixed stability fee for a term;
* Integration with Gnosis Auction Protocol V2 to discover rates and perform large sales competitively and transparently;
* Utilizes Asymmetric Demand: There is no requirement to find buyers for Zeros to be able to sell Claims.
