# Maker Fixed-rate Vaults

Deco protocol in collaboration with MakerDAO can offer fixed-rate vaults to Maker vault users. For vault owners, the process is as simple as purchasing a token which earns a yield that offsets the entire stability fee on a vault.

Vaults need not be migrated to an external protocol before or after the fixed-rate expires which allows Maker protocol to retain its core business and not lose stability fees.

Vault owners do not have to change any of their own vault management processes. The UI or smart contracts they use to manage a vault will continue to be used. Vault itself remains fully independent, and within Maker Protocol, while the vault owner enjoys a fully fixed-rate on their stability fee.

![Fixed-rate Vaults](_media/../../_media/maker-deco-diagrams.png ':size=600')

Usage steps are as follows,

* Vault owners simply purchase an additional token. Ex: 100,500 CLAIM-FEE-ETH-A tokens valid for three months to hedge their ETH-A vault with 100,500 DAI in debt for three months.
* Purchase price of the token is the total stability fee they would pay. Ex: Vault owner pays 500 DAI to purchase these 100,500 CLAIM-FEE-ETH-A tokens, uses the 100,000 DAI in debt for their own purposes, which fixes their total vault debt at 100,500 DAI for the entire duration the CLAIM-FEE-ETH-A tokens are valid for irrespective of the changes to the ETH-A stability fee over this duration.
* Token earns yield during its validity period which equals the actual stability fee accruing on the vault. Ex: ETH-A stability fee increases to 50% APR at some point which increases the vault debt to 113,000 DAI. 100,500 CLAIM-FEE-ETH-A tokens would earn 12,500 DAI as yield over the same duration(equal to the stability fee accrued) which will pay down debt back to 100,500 DAI.
* Vault owner can collect yield from token as many times as they want to continuously payback accrued stability fee on their vault and eliminate any additional interim exposure to a lower collateral ratio. Ex: Vault owner can collect Dai yield from the 100,500 CLAIM-FEE-ETH-A tokens as many times as they wish to repay the additional debt being continuously added to their vault with the real stability fee.
* Token balances are fungible and allow vault owners to hedge a portion of their debt by buying a lower amount of tokens than the vault debt. Ex: Vault owner can choose to purchase only 50,000 CLAIM-FEE-ETH-A tokens to hedge a portion of their vault debt.
* Vault owners can also sell the token on the market at any time. Ex: Vault owner can sell their 100,500 CLAIM-FEE-ETH-A tokens at any time based on the latest stability fee outlook.
* Vault owners can also purchase a new token to extend the hedge when their current tokens expire.

There are three main components to this integration all of which are under the control of MakerDAO,

* FEE token for each collateral type. Ex: FEE-ETH-A
* Deco instance deployed for each FEE token. Ex: Issues ZERO-FEE-ETH-A and CLAIM-FEE-ETH-A tokens.
* One generic ZERO-A collateral type setup to source Dai for FEE token issuances of all collateral types. Ex: ZERO-FEE-ETH-A, ZERO-FEE-WBTC-A, etc. can be locked as collateral in the ZERO-A collateral type to generate Dai used internally for the FEE token issuance.

## Fee Token

Fee tokens are setup to earn yield which tracks the stability fee of a collateral type within the Maker protocol like ETH-A. They are like CHAI token balances which in conjuction with the Pot contract have a yield which tracks the Dai Savings Rate. Their issuance is restricted and placed under the sole control of MakerDAO since the yield earned by the fee tokens (matching the stability fee) are its long-term liability in the financial sense.

Fee tokens are denoted with the token symbol `FEE` appended to the symbol of the collateral type. Ex: `FEE-ETH-A`. These tokens are only used for internal accounting purposes and are not allowed to circulate freely in the market and be directly held by individual users. All fee token issuances are immediately locked within Deco to ensure a maturity date is set right at their issuance. Deco is setup to automatically burn them after this maturity date which in turn removes the liability for MakerDAO having to pay the yield indefinitely.

In exchange for holding this liability, MakerDAO benefits by being able to collect the entire stability fee for the duration (like 3, 6, or even 12 months) up front from a vault owner versus slowly collecting a portion from the vault each block. From a usage perspective, this is a great method to achieve vault user lock-in for the duration since they would have paid their entire stability fee up front to MakerDAO.

Deco's novel safety feature, Close, ensures that all outstanding FEE token supply can be burnt and all long term obligations tied to future maturity dates do not become an impediment to the redemption process for Dai holders whenever emergency shutdown is triggered.

## Deco Instance

In this section, we will outline how the Deco protocol integration interfaces are modified to suit the needs of the Maker protocol.

MakerDAO and vault owners undertake the following steps,

1. MakerDAO creates a Dai balance, deposits it within FEE contract to mint the FEE token balance.
2. Locks FEE token within Deco to issue ZERO-FEE and CLAIM-FEE token balances.
3. Locks the ZERO-FEE token in the token adapter to receive a ZERO-A internal balance within the Maker protocol.
4. Generates debt 1:1 and uses the Dai to settle the originally generated Dai balance.
5. MakerDAO can auction the CLAIM-FEE token balance for vault owners and collect the Dai they pay as stability fees.
6. Vault owner collects yield on CLAIM-FEE token until maturity to offset the stability fee accruing on their vault with no other impact elsewhere.
7. CLAIM-FEE token balance is burnt automatically when the yield for the entire duration is collected.
8. ZERO-FEE token balance will be redeemed for the same amount of Dai at maturity, or emergency shutdown to cancel the entire debt from the ZERO-A collateral type and burn both ZERO-FEE and FEE token balances and remove them from circulation.

We will now outline how the base Deco protocol is modified to deploy instances that handle the needs of this integration with the Maker protocol.

### Snapshot

`snapshot` trustless function is setup to read the fraction value from the FEE token contract on-chain. As the stability fee of a collateral type increases, the yield of the FEE token would increase correspondingly which reflects in a lower fraction value recorded and stored over time within Deco.

`insert` is implemented to allow MakerDAO to accurately insert fraction values at maturity timestamps.

### Issuance

MakerDAO first generates the Dai needed to perform this issuance process. We will explain how this Dai generated is fully covered by a corresponding amount of debt right at the end of the issuance process.

`issue` is placed under the control of MakerDAO.

For ex: Fraction value at issuance timestamp is `0.9`. MakerDAO mints 9 Mil FEE-ETH-A tokens, executes `issue` for a maturity date. This would create 10 Mil ZERO-FEE-ETH-A and 10 Mil CLAIM-FEE-ETH-A token balances (Their notional amounts track 10 Mil because 9 Mil FEE-ETH-A tokens when fraction value is 0.9 are worth/can claim 10 Mil Dai).

### CLAIM-FEE Token Sales

MakerDAO can convert the CLAIM-FEE-ETH-A balance it holds at the end of this issuance process to ERC20 and sell them to interested vault owners through auctions using Gnosis Protocol v2.

Vault owners execute the `collect` function periodically (or just once after maturity) to collect yield earned in Dai and use the balance to repay additional debt accrued on their vault due to the stability fee of the collateral type.

### Settlement Functions

Deco typically settles its obligations in the yield token balance itself and in most integerations the settlement function implementations are not modified. For this integration, Deco would have to perform a FEE token redemption as well since FEE tokens cannot be distributed directly to zero and claim holders and allowed to freely circulate. All settlement functions like `redeem`, `collect`, `withdraw`, `cashZero` and `cashClaim` are setup to redeem FEE tokens for a Dai balance which is then distributed to the holders. This would not break any guarantees since FEE tokens are backed by MakerDAO, can never be insolvent by definition, and can be redeemed for Dai in any scenario. 

## ZERO-A Collateral Type

New unbacked dai is minted at the start of the issuance process which is repaid immediately by generating an equal amount of debt using the ZERO-FEE tokens themselves as collateral within the Maker protocol.

One point to note here is that a single `ZERO-A` collateral type can handle the needs of Fee tokens setup for all collateral types.

`ZERO-A` is backed 1:1 with any ZERO-FEE token. Ex: 10 Mil ZERO-FEE-ETH-A tokens would be locked as collateral to generate 10 Mil Dai, and immediately repay the exact amount of Dai used at the start of the process to issue 9 Mil FEE-ETH-A tokens also worth 10 Mil Dai at a fraction value of 0.9.

The collateral type need not be setup with price feeds or liquidation infrastructure.

Ex: After 10 Mil ZERO-FEE-ETH-A tokens mature, they can be redeemed for their notional amount which is 10 Mil Dai, which will be able to repay the entire debt and retire the vault as well. If emergency shutdown is triggered, the same redemption will happen irrespective of the maturity date and the dai can be used as part of the redemption process.

### Token Adapter

The token adapter setup to create the internal `gem` balance for the `ZERO-A` collateral type accepts an approved list of MakerDAO managed FEE tokens.

`join` transfers the ZERO-FEE-ETH-A token balance to the adapter and issues the `ZERO-A` internal gem balance to the depositor, which would be MakerDAO.

Token adapter would implement public functions that allow anyone to execute Deco redemption functions on ZERO-FEE token balances it holds. Executing `redeem` replaces the ZERO-FEE token balance the adapter holds with an equal amount of Dai.

`repay` function is implemented within the adapter to use any Dai balance it holds to repay the `ZERO-A` vault debt and remove both debt and Dai from circulation.

`sell` function can be implemented optionally to allow MakerDAO to trigger an auction of the ZERO-FEE tokens it holds for Dai at a discount to both incentivize long-term Dai holders as well as retire the ZERO-A vault debt early with the sale proceeds.

Deco instance would at some point always replace the ZERO-FEE balances held by the token adapter contract with Dai. This Dao is then used to payback all debt.

`exit` is not normally used at all, but it is setup to exit the internal ZERO-A collateral balance for Dai instead of the individual ZERO-FEE token balances that were original deposited.

## Emergency Shutdown and Close

If emergency shutdown is triggered, `calculate` function is setup to forward all Dai to Zero-FEE tokens irrespective of the maturity date. All ZERO-FEE token balances held by the adapter would be immediately replaced with Dai. This allows the internal ZERO-A token balance to be converted to Dai and redeem the collateral owed.

To remove additional steps to redemption, Deco Fee instances can be shutdown to retire the debt in ZERO-A collateral type before emergency shutdown in Maker protocol is triggered.

## Summary

Almost all fixed-rate protocols have faced liquidity issues because they have to find buyers for both zero and claim tokens at the same time after the conclusion of the issuance process. By integrating with a collateral type, we have removed the need to find buyers for zero balances immediately. With the process we have engineered, the DAO can temporarily or permanently hold ZERO tokens which creates a seamless user experience for MakerDAO to satisfy the demand for CLAIM-FEE tokens without constraints whenever it emerges.

ZERO-FEE tokens can also be unlocked from the adapter whenever demand emerges. If not, the debt is setup to be automatically repaid after maturity date, or if emergency shutdown is triggered, or if MakerDAO wants to shut down the Deco instance for whatever reason. Auctioning Zeros will help MakerDAO discover the yield curve for Dai and can use the information to set DSR effectively.

To summarize the benefits of this integration to vault owners and MakerDAO,

* Fully fixed stability fee with no upper limits. Once purchased, the token issued by Deco and MakerDAO can offset the stability fee up to any increase.
* Vault owners do not have to make any changes to their vault ownership or management.
* Opens up an additional source of profits in the form of `fixed-rate premium` for MakerDAO and gives it the ability to upsell a new product to the largest vault owners and future RWA vaults who also tend to borrow at scale.
* Vault owner stickiness is vastly improved, especially for the largest vaults, since they have prepaid their stability fees as well as locked in a fixed stability fee for a term.
* Integration with Gnosis Auction Protocol V2 to discover rates and perform large sales competitively and transparently.
* Need not find buyers for Zeros issued to be able to sell Claims.
* For the first time, MakerDAO has a tool to discover the stability fee vault owners are willing to pay by analyzing the CLAIM-FEE token sale data.
