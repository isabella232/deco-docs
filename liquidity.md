# Liquidity

Zero and claim balances can trade on a wide variety of exchanges including AMMs because they are compatible with both ERC20 and ERC721 token standards.

We have developed the following liquidity mechanisms to give claim holders additional choices in how they manage their claim balances.

## Early Collect

Claim holders have the option to collect yield earned so far early without having to wait until maturity. The original claim balance is removed from supply and a new claim balance with an updated issuance timestamp is issued back to the user along with the yield earned after `collect` is executed.

![Early Collect](_media/collect.png ':size=600')

## Rewind

Claim holders can move their issuance timestamp to an earlier one by sending the Deco instance additional yield token balance.

`collect_` timestamp has to fall before `issuance` timestamp, and `maturity` timestamp would remain the same.

```solidity
// Rewinds issuance of claim balance back to a past timestamp
function rewind(address usr, uint256 issuance, uint256 maturity, uint256 collect_, uint256 bal_) external;
```

Rewind allows claim holders with a non-standard issuance timestamp to move it to one that is a market standard and take advantage of the additional venues for liquidity it might enjoy. It also allows those who have accidentally collected yield to set the issuance timestamp back to the original one.

## Slice, Merge, Activate

Users holding a long-range claim balance that could mature a year later, can slice the balance into two claim balances covering contiguous time periods and hold/manage/trade them independently with `slice`.

![Slice](_media/slice.png ':size=600')

`t2` is the slice timestamp. `t1` and `t3` are the original claim balance issuance and maturity timestamps.

```solidity
// Slices one claim balance into two claim balances at a timestamp
function slice(address usr, uint256 t1, uint256 t2, uint256 t3, uint256 bal_) external;
```

Any two contiguous claim balances, sliced or otherwise, can be merged together into one claim balance as well with `merge`.

`t2` will be the maturity timestamp of the first claim balance, as well as the issuance timestamp of the second claim balance.

```solidity
// Merges two claim balances with contiguous time periods into one claim balance
function merge(address usr, uint256 t1, uint256 t2, uint256 t3, uint256 bal_) external;
```

Sliced balances have no guarantee that the issuance timestamp set on the second part would have a fraction value set in the future. `activate` is able to move the issuance timestamp (slightly) to a new timestamp that has a valid fraction value set, thus allowing the claim balance to be able to collect the yield earned from then until maturity.

`t1` will be the original issuance timestamp of the sliced balance which cannot collect yield because a fraction value does not exist at it. `t2` is a timestamp that comes after and has a fraction value set.

```solidity
// Activates a balance whose issuance timestamp does not have a fraction value set
function activate(address usr, uint256 t1, uint256 t2, uint256 t3, uint256 bal_) external;
```
