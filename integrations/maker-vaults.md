# Maker Fixed-Rate Vaults

Deco Protocol integration with the Maker protocol offers fixed-rate vaults to existing Maker vault users by allowing them to hold a `CLAIM-FEE` balance on Deco which earns yield in Dai to offset the stability fee being added to a regular vault in a certain time period.

A Vault need not migrate to an external protocol, either before or after the fixed rate expires. Vault owners may continue to use the same UI they normally use to manage their existing vault. Deco integration deployed for the Maker protocol is controlled and maintained by Maker governance, allowing the DAO to control fixed rates being offered without any external dependencies.

There are two main components in this integration:

* `dss-gate` to set an approval limit on dai drawn from `vat.suck()`
* Deco protocol instance to issue `CLAIM-FEE` balances for all valid collateral types

## Usage Steps

1. The vault owner purchases a `CLAIM-FEE` balance, issued for the collateral type which matches their existing vault, for a duration they would like to fix the stability fee. Ex: 100,500 `CLAIM-FEE-ETH-A` balance valid between `01/01/2022` - `03/31/2022` can fix the stability fee on an ETH-A vault with 100,500 DAI in debt for three months.
2. Total stability fee paid for this duration is the purchase price of the `CLAIM-FEE-ETH-A` balance. Ex: 2,500 DAI for 100,500 `CLAIM-FEE-ETH-A`.
3. `CLAIM-FEE-ETH-A` balance pays back a yield(in dai) equal to the variable stability fee accruing on the vault to offset it. Ex: ETH-A stability fee increases to 50% APR at some point which increases the vault debt to 113,000 DAI. 100,500 CLAIM-FEE-ETH-A would earn 12,500 DAI as yield over the same duration (equal to the stability fee accrued) which will pay down debt back to 100,500 DAI.
4. Both yield and vault debt amounts compound at the same rate when they have the same notional amount at the issuance timestamp. Vault owner can collect yield any number of times to payback the variable rate debt being accrued and to keep their collateralization ratio high.
5. CLAIM-FEE balances are fungible which allows vault owners to adjust their balance to match the changes to the debt amount on their vault.
6. Vault owners can also purchase new CLAIM-FEE balances as the current one is about to expire to extend their fixed rate.

## Integration Customization

The following changes were made to the base Deco Protocol to better support the needs of the Maker protocol integration:

* `ilk` collateral type is encoded within the `class` value along with issuance and maturity timestamps. This allows Maker governance to issue claims for all collateral types from a single deployed Deco instance and reduces operational overhead by not having to maintain multiple deco instances.
* Notional amounts in the base token(like Dai) are used for settlement instead of the "normalized" yield token balance(like the FEE token or CHAI).
* `frac`(inverse of rate) changed to `rate` since balance accounting is in notional amounts.
* Issuance is restricted to only claims, not zeros and claims.
* Issuance controlled by Maker governance.
* Both the public `snapshot` and governance controlled `insert` is made available to store historical `rate` values.
* Deco instance can be closed by anyone after emergency shutdown of the Maker protocol, or by Maker governance at any time.

### Rate History

Historical `ilk.rate` values are critical to the issuance and settlement of CLAIM-FEE balances and are stored in the `rate` mapping for all collateral types.

Rate functions are customized in the following manner:

`snapshot` allows anyone to store the current `rate` value of a collateral type. Please note that deco will not execute `jug.drip()` to update the rate value within vat for the current timestamp.

`insert` allows Maker governance to insert a `rate` value at an arbitrary timestamp for use in situations where a snapshot could not be captured. We have encoded a few precautions to safeguard users,

* insert timestamp cannot be in the future, it has to be before the latest rate value captured by a public snapshot.
* a timestamp that falls before the insert timestamp with a valid rate value present is required to be used as a guardrail to prevent governance from inserting an arbitrary rate value.

### Collateral Type Initialization

`initializeIlk` checks whether the `ilk` value is initialized and valid within the Maker protocol.

A `rate` value snapshot is recorded and the `initializedIlks` bool value is set to `true`.

All valid ilks within the Maker protocol need not be supported since only Maker governance can initialize them within Deco. Once initialized, an ilk cannot be uninitialized or removed later, but Maker governance in is control and can choose to stop new issuances.

### Issuance

`issue` is executed by Maker governance to create new claim balances. At issuance, each `CLAIM-FEE` balance is tied to a specific `issuance` timestamp and a `maturity` timestamp along with an `ilk` collateral type to identify the stability fee it will track and yield.

`CLAIM-FEE` balance needs to be match the total debt(notional amount) of a vault(`urn`) based on the rate value at the issuance timestamp, `urn.art * ilk.rate`

### Settlement

`collect` allows `CLAIM-FEE` balance holders to withdraw yield earned from the `issuance` timestamp until a `collect` timestamp they select. This allows balance holders to withdraw yield earned any number of times instead of just once after maturity.

Yield is transferred in the form of Dai, and a new `CLAIM-FEE` balance is also issued with the issuance timestamp updated to `collect` timestamp to allow balance holder to claim future yield after collect. When `collect` timestamp equals `maturity` timestamp, once governance is able to insert a rate value, full yield amount is passed to the balance holder and the entire `CLAIM-FEE` balance is burnt.

`rewind` allows a `CLAIM-FEE` balance holder to set the current `issuance` timestamp back to a past timestamp by paying back the yield they have collected. This is helpful to make a balance fungible with another older balance class.

## Gate

Proceeds from the sale of a `CLAIM-FEE` balance after issuance are forwarded to the surplus buffer. MakerDAO gets to collect stability fees upfront for an entire term which locks the vault owner for the period. In exchange, Maker governance allows Deco to offset the variable stability fee over the time period on a vault by passing the charged stability fee on a vault back as yield. Deco does not access `vat.suck()` directly, instead Maker governance sets up a new gate contract and authorizes Deco to access `gate.suck()` which can also be used to draw Dai. Gate is managed by Maker governance and adjusts limits periodically on the Dai amount Deco needs to draw.

## Emergency Shutdown and Close

Deco instance can be closed by Maker governance to settle all current and future obligations instantly and stop future `CLAIM-FEE` issuances of all collateral types. Any address can trigger close after Maker protocol undergoes emergency shutdown.

`close` sets `closeTimestamp` to the `block.timestamp` at its execution. `closeTimestamp` is set to a very large value `MAX_UINT` at deployment until `close` is executed to prevent `calculate` and `cashClaim` functions from being activated until the deco instance is closed.

`calculate` allows Maker governance to set the valuation of `CLAIM-FEE` balances when their maturity timestamps fall after the `close` timestamp. This allows Maker governance to payout the residual value of the unused `CLAIM-FEE` balance in Dai.

`cashClaim` allows `CLAIM-FEE` holders to exchange their unused balance for Dai. For ex, let us consider that a `CLAIM-FEE-ETH-A` balance exists with `issuance` set to `01-01-2022 12:00:00 UTC`, maturity set to `03-31-2022 12:00:00 UTC`, and the Deco instance is closed on `02-01-2022 12:00:00 UTC`. `CLAIM-FEE-ETH-A` holder can execute `collect` to withdraw their yield until the latest rate value captured for ETH-A, and can execute `cashClaim` to withdraw an amount of Dai based on the valuation that governance sets for the residual time period on the `CLAIM-FEE` balance until maturity. A backup balance is required in the gate contract for `cashClaim` to be successful after emergency shutdown of the maker protocol.

## Slice

We have incorporated a few functions to allow `CLAIM-FEE` holders to manipulate their balances and offload a portion of the time period on the balance to others if needed.

`slice` allows a user to split the time period of their `CLAIM-FEE` balance at a timestamp between its `issuance` and `maturity` and create two new `CLAIM-FEE` balances which can be independently managed and transferred.

`merge` allows a user to combine two contiguous `CLAIM-FEE` balances into a single balance.

`activate` allows a user to slightly move the `issuance` timestamp of a previously sliced `CLAIM-FEE` balance forward to one which has a valid rate value snapshot.

## Benefits

We have designed and developed an integration with the Deco protocol to allow MakerDAO and its vault owners to derive the following benefits:

* Fully fixed stability fee for vault owners: Offset the stability fee up to any increase;
* No Vault Management Changes: Vault owners do not have to make any changes to how they own or manage their existing vault;
* New Revenue Stream: This integration opens an additional source of income in the form of fixed-rate risk premiums for MakerDAO and gives it the ability to upsell existing users a new feature, especially to the largest vault owners;
* Vault owner retention: Long term vault owner stickiness is vastly improved, especially for the largest vaults, since they have prepaid their stability fees and are locked into a fixed stability fee for a term;
* Ability to discover fixed-rates independently;

## Governance

The following MIPs were approved by MakerDAO to form the Deco Fixed-Rate Core Unit and support this integration,

* [MIP39c2-SP23: Adding the Deco Fixed Rate Core Unit](https://mips.makerdao.com/mips/details/MIP39c2SP23)
* [MIP41c4-SP24: Facilitator Onboarding, Deco Fixed Rate Core Unit](https://mips.makerdao.com/mips/details/MIP41c4SP24)
* [MIP40c3-SP35: Deco Fixed Rate Core Unit Budget](https://mips.makerdao.com/mips/details/MIP40c3SP35)
* [MIP40c3-SP36: Deco Fixed Rate Core Unit MKR Budget](https://mips.makerdao.com/mips/details/MIP40c3SP36)
