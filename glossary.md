# Glossary <!-- {docsify-ignore-all} -->

## A

### approvals

Approvals allow users to delegate full management of their Deco balances to other Ethereum addresses.

## B

### balance adapters

Balance adapter smart contracts allow Deco balances to conform to either ERC20 or ERC721 NFT token standards based on user choice.

### base asset

Token deposited into a yield protocol.

### blocktime

Timestamp at which a block is mined in a blockchain.

## C

### cash

Cash functions are activated after a Deco instance is closed to allow users to imme- diately settle their Deco balances at future maturity dates for yield token balances.

### CLAIM-CHAI

Claim balance in a Deco instance set up for CHAI. CLAIM-CHAI balance is created when a user executes the issue function on a Deco instance deployed for the CHAI protocol.

### class

Deco balances are separated with a class value derived from their issuance and matu- rity dates to only allow fungibility among balances that share the same characteristics.

### close

Close function stops new Zero and Claim issuance and allows existing Zero and Claim balances for all maturity dates, past or future, to be immediately settled for their share of the yield token deposits held by Deco.

### collect

Collect allows Claim balance holders to withdraw the yield earned on their balance.

## D

### DAI 

Stablecoin which is pegged to USD issued by the Maker protocol.

### Dai Savings Rate

Yield protocol managed by the Maker protocol to give all DAI holders additional yield when they lockup.

## E

### emergency shutdown

Maker protocol feature which stops all new issuance of Dai and allows holders to redeem it for various amounts of collateral held within.

## F

### fixed rate

An exact amount of yield earned by holders for a pre-defined time period on their deposit.

### fraction

Fraction value is the amount of yield tokens a unit of the base asset can purchase. For example, the amount of CHAI one DAI can purchase. It is typically 1 or lower since the yield token almost always is worth more in terms of the base asset due to the additional yield it accrues over time in the base asset itself.

## G

### governance

Body of users who use coordination methods like voting, delegation, et cetera to take actions they are authorized to on a Deco instance.

## I

### insert

Insert allows governance to input fraction values for various timestamps to be used for settlement purposes of balance holders.

### instance

A Deco instance is a deployment of the Deco protocol smart contracts to handle the needs of a specific yield token.

### issuance

Users generate Zero and Claim balances by making a yield token deposit into a Deco instance.

## L

### leverage

Ability to lock a token balance as collateral and borrow another asset as debt to be repaid back later.

### liquidity provider

Users who deposit base assets into protocols that in turn provide various services like exchange, staking, lending, et cetera to other end-users.

## M

### Maker protocol

Decentralized lending protocol which allows users to borrow against a variety of collateral and generate the USD pegged stablecoin DAI.

### market maker

User who specializes in supplying trading liquidity for a variety of assets on exchanges.

### maturity

Timestamp after which assets can be exchanged back for their promised shares of the locked yield token balance from Deco.

## P

### price

Value of the yield token in base asset terms. For example, the amount of DAI a CHAI balance can withdraw from the CHAI protocol.

## R

### redeem

Zero balances are redeemed for their share of the yield token deposit in Deco after their maturity date.

### rewind

Issuance of any Claim balance can be set to a previous timestamp provided a fraction value snapshot is available at that timestamp. This is typically used to adjust a Claim balance from a non-standard issuance date to a standard date that exists in the past to take advantage of available ERC20 token adapter and exchange liquidity.

## S

### settlement

Process of giving back the entire yield token balance deposited at issuance back to Zero and Claim holders based on the terms attached. Zero and Claim balance holders use functions like redeem, collect, withdraw, or cash based on the circumstance for settlement.

### slice

Splitting a Claim balance at a timestamp between its issuance and maturity times- tamps.

### snapshot

Snapshot function captures the fraction value of a yield token at the current timestamp and stores it for future reference during settlement.

### solvency

Yield protocols are considered solvent when they can honor redemption of all base asset deposits and accrued yields.

## T

### term

Time between issuance and maturity timestamps.

### trustless

Trustless functions in Deco instances can capture data directly from the yield protocol’s smart contracts and allow anyone to execute them.

## W

### withdraw

Function which allows early redemption of a Zero balance before its maturity timestamp.

## Y

### yield

Amount of additional base asset balance accrued after a certain duration which the original deposit can claim from the yield protocol.

### yield protocol

Protocol which takes in a base asset deposit from liquidity providers and passes them a yield generated from the fees charged to users for providing a service.

### yield token

Yield protocols give liquidity providers a yield token in exchange for their base asset deposits. Yield token balances can typically be exchanged back again for the base asset deposit as well as any additional yield earned by it from the yield protocol.

### yield-only

Asset that promises its holder only the yield earned at settlement.

## Z

### ZERO-CHAI

Zero balance in a Deco instance set up for CHAI. ZERO-CHAI balance is created when a user executes the issue function on a Deco instance deployed for the CHAI protocol.

### zero-coupon

An asset that trades at a discount to its face value for which it settles at on the maturity date and without making any explicit yield payments to the holder during the term.
