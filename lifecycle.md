# Core Lifecycle

Core lifecycle of Deco consists of issuance (mint), which brings two new assets into circulation, zero and claim, and settlement (burn), which removes them both from circulation. Asset balances are always attached to a maturity timestamp at issuance which establishes a benchmark future settlement timestamp.

Deco manages all its internal accounting using the yield token balance. At issuance, users exchange their yield token balance for an amount of zero and claim balances which can be freely transferred to others or traded on various markets. Later, the balance holders are also able to exchange these two assets back for their due portion of the yield token deposit and accrued yield.

How the original yield token balance deposit gets split between zero and claim balance holders after maturity is decided based on the observed yield (change in price of yield token) earned by the yield token balance between issuance and settlement.

![With or Without Yield after Issuance](_media/zcnoyield.png ':size=600')

If a yield is high, the zero will receive a smaller proportion of the original deposit, if the yield is low, the zero will receive a larger portion.

## Lock and Unlock

Lock and Unlock are internal functions and are not user facing. All functions that transfer a yield token balance into Deco utilize `lock` while the functions that transfer a yield token balance out of Deco utilize `unlock`.

Lock outputs the `notional amount` value(zero and claim balance amounts issued), based on two inputs- `frac_` fraction value, and `tbal_` yield token balance user transfers to the Deco instance.

Unlock transfers a yield token balance from the Deco instance to the zero or claim holder which settles their `notional amount` balance appropriately after maturity.

```solidity
// Lock transfers yield token balance from user to deco
function lock(address usr, uint256 frac_, uint256 tbal_) internal returns (uint256);

// Unlock transfers yield token balance from deco to user
function unlock(address usr, uint256 frac_, uint256 bal_) internal;
```

Both functions are designed to automatically adjust for the decimals of the yield token while normalizing the notional amount value to `18` decimals. The `if-else` statements can be removed prior to deployment since the exact decimals of the yield token that the Deco instance will be attached to would be known.

![Core Lifecycle](_media/corelifecycle.png ':size=600')

## Issuance: Zero and Claim

Zero and claim balances are issued simultaneously to a user who executes `issue` and successfully transfers a yield token balance to a Deco instance.

```solidity
// Issues zero and claim balances in exchange for yield token balance
function issue(address usr, uint256 issuance, uint256 maturity, uint256 tbal_) external;
```

A fraction value has to exist at the exact `issuance` timestamp for it to succeed. `maturity` can be set to any timestamp in the future, but we recommend users to follow the standard maturity timestamps that the Deco instance operator has published for the monthly, quarterly, or yearly series.

`tbal_` is the amount of yield token balance being transferred to the Deco instance. The lock function would calculate the `notional amount` value which is used to mint zero and claim balances.

## Settlement: Zero

After maturity, zero holders can settle their balance to receive a yield token balance from the Deco instance by executing `redeem`. They will have to redeem the yield token balance for the base asset from the yield protocol by themselves, which should match the notional amount when the yield protocol has no solvency issues and can honor redemption. Ex: 100 ZERO-CHAI redeems enough CHAI which to be able to unlock 100 DAI.

```solidity
// Redeems a zero balance after maturity
function redeem(address usr, uint256 maturity, uint256 collect_, uint256 bal_) external;
```

`maturity` timestamp is used to calculate the `class` value of the zero balance to redeem.

In the worst case, the zero balance holder would `collect_` to the publicly known `latestFracTimestamp` value to receive the exact notional amount in base asset they are owed after maturity through the yield token balance they receieve. Ex: 100 ZERO-CHAI token maturing on July 31st will be able to redeem enough CHAI at any time after to receive 100 DAI.

In the best case, the zero balance holder can set `collect_` to the `maturity` timestamp, or to another one right after with a recorded `frac` value, to finally receive both the notional amount as well as the yield it has earned from maturity (or collect timestamp) until the latest frac timestamp recorded at redemption. This ensures zero holders who forget to settle do not lose any yield earned irrespective of when they actually redeem their zero balance from the Deco instance. Ex: 100 ZERO-CHAI token maturing on July 31st, and redeeming it on December 31st, by supplying a `collect_` timestamp right after July 31st will be able to redeem enough CHAI to receive 100 DAI as well as the yield generated by the 100 DAI from the `collect_` timestamp until December 31st. Savings generated after maturity by the Zero balance are not lost at all if the user takes care to supply the correct fraction records for Deco to be able to do its settlement calculations.

![Deposit to Withdrawal: Zero](_media/dtwzero.png ':size=600')

## Settlement: Claim

Similarly, claim holders settle their balance after maturity by executing `collect`. Deco will send the claim holder enough yield token balance which they can use to redeem enough in base asset from the yield protocol to match the yield earned between issuance and maturity timestamps.

```solidity
// Collects yield earned by a claim balance
function collect(address usr, uint256 issuance, uint256 maturity, uint256 collect_, uint256 bal_) external;
```

When `collect_` input value is set to the `maturity` timetamp at which a fraction value is present, the entire yield earned, as well as the yield this balance would in turn earn starting at maturity until the collect transaction is also paid out.

Collect need not be executed only after maturity. It can be executed any number of times before maturity as well once there is some yield available to collect. This gives the holder flexibility to either collect the yield early, or leave it invested in the yield protocol. To ensure the collected yield is accounted for, the issuance timestamp of the claim balance would be modified to the timestamp until which the yield was collected so that the balance can be distinguished from other balances that have not collected, as well as to not payout this yield again in the future.

In Deco instances not operated by a governance mechanism, at least one user has to execute `snapshot` to capture a fraction value right before maturity to allow all claim balance holders at that maturity timestamp to collect their yield at any time in the future.

![Deposit to Withdrawal: Claim](_media/dtwclaim.png ':size=600')

A governance mechanism can insert a fraction value precisely at the `maturity` timestamp for the maximum benefit of both zero and claim holders who can redeem the maximum amounts with the entire additional yield earned until whenever they execute settlement in the future.

Without governance involved, zero holders have to time their snapshot transactions to be mined right after maturity to claim additional yield later, and claim holders have to time their snapshot transactions to be mined right before maturity to be able to collect the maximum yield possible. While we have shown how Deco instances can be setup to work in a fully trustless manner, we can also see how governance mechanisms can play a crucial role in ensuring Deco instances remain safe and ensure users do not incur losses by inserting fraction values precisely at the maturity timestamps.
