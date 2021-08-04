# Fraction Values

Yield protocols gives users back a static balance of yield tokens that does not change. But the amount of base asset deposit the static yield token balance can claim increases with time as more yield is earned by the yield protocol.

We can see this in action in the below diagram, which shows how a static yield token balance (95.23 CHAI) that was given to the user in exchange for their 100 DAI deposit into the Dai Savings Rate (DSR) smart contract in 2020. The yield earned by DSR grows at various rates over time to allow them to withdraw a larger amount of 120 DAI at withdrawal in 2023.

![Yield Token Compounding](_media/yt.png ':size=600')

Fraction value is the amount of yield token balance (CHAI) a user receives upon depositing a unit of base asset(1 DAI) into the yield protocol. In the above example, the fraction value at deposit time would be `0.9523` since the user received 0.9523 CHAI for 1 DAI deposited into DSR.

Fraction value at the genesis of any yield protocol would start at `1` and drops lower as it accrues yield continuously over time.

![Deposit to Withdrawal: Yield Token](_media/dtwyieldtoken.png ':size=600')

## Fraction Value Storage

Yield tokens typically do not store historical yield information on-chain, but historical fraction values are imperative to settling fixed maturity assets that were issued in the past at some future timestamp. In Deco, the `frac` mapping stores fraction values captured at various timestamps by the Deco instance. A record of the last fraction value captured by the instance is kept as well by storing the latest capture timestamp in `latestFracTimestamp`.

```solidity
mapping(uint256 => uint256) public frac; // frac timestamp => frac value [wad] ex: 0.85
uint256 public latestFracTimestamp; // latest frac timestamp
```

Capturing and storing fraction values in the `frac` mapping at various timestamps for future reference is a key part of the issuance and settlement process. All Deco and yield token integrations have to implement at least one of two functions- `snapshot` and/or `insert`.

## Snapshot

We recommend developers implement the public `snapshot` function to allow anyone to capture and store the current fraction value when executed. Thisc can be done for yield protocols where it is readily available to read or calculate from on-chain. Ex: Dai Savings Rate, Compound Tokens, Aave Tokens, Yearn Tokens, etc. report the price value in various forms which can be used to compute the fraction by taking an inverse.

```solidity
// Captures fraction value of a yield token in a trustless manner for current timestamp
function snapshot() external;
```

Please note that `snapshot` is customized for the Deco implementation of each yield protocol to capture the fraction value from the right source and transform it if necessary.

## Insert

When a yield protocol does not make the fraction value readily available on-chain, it will be computed off-chain by a trusted governance mechanism and then stored for a timestamp. Developers can implement the restricted `insert` function to allow appropriate governance mechanisms to insert the fraction values at various timestamps based on the Deco instance needs. Insert can also implemented in addition to Snapshot to act as a backstop to handle scenarios where users have failed to capture snapshots themselves, and also to insert fraction values at precise timestamps to allow accurate settlement without losses.

```solidity
// Governance inserts a fraction value at a timestamp
function insert(uint256 t, uint256 frac_) external onlyGov;
```
