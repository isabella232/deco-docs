# Safety

The philosophy underpinning the design of Deco protocol is that users must always have access to a withdrawal mechanism, and that the underlying yield token should never be locked by default into the maturity timestamp. Maturity timestamp is a characteristic of a token that gives it a valuation, not a timestamp until which the deposits have to be locked up.

Fixed-term asset holders should not be exposed to additional risk when their yield token balances are locked until a maturity timestamp. They will not be able to confidently participate in longer duration issuances as well when options to redeem their yield token balance early are not present. It would also automatically expose them to additional risk as compared to regular yield token holders who are able to redeem their base assets at any time and exit from the yield protocol leaving fixed-term asset holders who are unable to do so accrue losses in the yield protocol.

Current protocols issuing yield tokens are unable to address the negative impact on users of having their yield tokens locked in for an extended period of time in fixed-rate protocols without the ability of redemption prior to maturity. The issuance of zeros and claims does not create liquidity issues for yield protocols because Deco provides this flexibility.

Deco has developed two important safety mechanisms to mitigate these issues,

* `withdraw`, a user-level safety mechanism, allows users themselves to settle their zero and claim balances early before maturity and withdraw the yield token balance deposited within Deco at any time without the involvement of the operator.
* `close`, a protocol-level safety mechanism, allows the Deco instance operator to shut it down and allow all zero and claim balance holders to withdraw a fair share of yield token balance immediately.

## Withdraw

Zero balance holders can purchase an equal amount of claim balance with the same maturity timestamp from the market to execute `withdraw` and transfer the yield token balance from the Deco instance early and before the actualy maturity timestamp. Deco can make early withdrawal work by simply allowing users themselves to take an equal amount of zero and claim balances of the same maturity timestamp out of circulation and withdraw the entire yield token balance deposited backing them.

![Withdraw](_media/withdraw.png ':size=600')

`withdraw` only takes in the `maturity` timestamp of both zero and claim being removed from circulation as input.

The function expects the user to `collect` all the yield earnt on the claim balance from its issuance until the recorded `lastFracTimestamp` or purchase a claim balance on the market with no collectable yield whose issuance is already set to `lastFracTimestamp`.

`bal_` is the amount of zero and claim balance the user performing withdrawal holds in their address.

```solidity
// Withdraws zero and claim balances before their maturity
function withdraw(address usr, uint256 maturity, uint256 bal_) external;
```

This behavior can be removed if desired by the operator by finding a way to permanently lock one of zeros and claims from circulation to force the other asset to stay in circulation, or by removing the function from the implementation of the integration with the yield protocol. These however can be sidestepped by our protocol-level safety mechanism, `close`.

## Close

There are several unexpected and troublesome scenarios that can occur in a yield protocol underlying a Deco instance. These problematic scenarios can be technical migrations, instability, or even protocol losses causing a complete shutdown, et cetera. While the Deco early withdrawal mechanism can help users to co-ordinate and withdraw on their own, ideally both zero or claim holders should also be able to independently withdraw a reasonable yield token amount due based on their current valuations, without the need to purchase the other asset they do not have (zero or claim) in their wallets under extraordinary circumstances.

### Trigger

Close can be triggered on a Deco instance by anyone when there is a clear on-chain signal demonstrating that the yield protocol is at risk. Governance can also be given authorization to trigger `close` on a Deco instance when it deems appropriate.

`closeTimestamp` is updated from the default `MAX_UINT` value where all maturities fall before it, to the current `latestFracTimestamp` which separates all zero and claim balances with a maturity timestamp after to be settled under a different process.

```solidity
// Closes this deco instance
function close() external;
```

Please note that the `close` implementation can be fully customized based on the needs of the yield protocol.

### Modifiers

Deco functions are not completely frozen when a Deco instance is closed by the operator. Instead the regular functions are allowed to operate as usual on all balances whose maturity timestamps fall before `closeTimestamp` because the fraction values they would need for a regular settlement are already available.

Zero balance that has matured before close but was never redeemed can continue to use `redeem`. Claim balances with yield earned before close can continue to use `collect`. You will notice that these functions have the `untilClose` modifier on them with the relevant timestamp passed to it to perform the check.

Zero and claim balances whose maturity timestamps fall after close timestamp are settled with special functions who can only operate on such balances and have the `afterClose` modifier.

### Future Maturity Valuation

After close, Deco does not simply give zero asset holders the entire notional amount back, even when it seems like this is the right approach, since the future yield that can be earned is essentially going to drop to nothing for claim holders. If we did that, this would incentivize zero holders to first buy their own balances at a discount to face value, and then try to immediately shut down Deco or even the underlying yield protocol in the hope of retrieving the full-face value without waiting until the maturity timestamp.

The Close mechanism has been carefully designed to remove the possibility of such early settlement attacks by zero asset holders. Deco instances will either encode a fair future valuation mechanism or lean on governance to input this after close to pass a fair value back to both zero and claim holders. This allows users to re-establish similar positions in an equivalent yield protocol or a new deployment of the same yield protocol.

Deco can only split the yield token balance it has on deposit for each future maturity timestamp between both zero and claim balances with maturity set to that timestamp.

`calculate` function needs to be implemented with an on-chain valuation calculation for assets with a future maturity timestamp, or execution control can be handed over to governance to input the ratio value for each future timestamp with issued zero and claim balances.

`ratio` mapping holds the yield token deposit split ratio values, which have to be between `0` and `1`. For ex, a ratio of `0.985` gives 98.5% of the locked yield token balance reserved for the maturity timestamp to zero balance holders and 1.5% to claim balance holders at the same maturity timesatmp.

Once a `ratio` value is present for a timestamp, the `zero` and `claim` view functions are able to report the yield token balance that can be redeemed by zero and claim balances of that maturity timestamp to `cashZero` and `cashClaim` functions.

```solidity
// Stores a ratio value
function calculate(uint256 maturity, uint256 ratio_) public;

// Determines value of zero with maturity after close timestamp
function zero(uint256 maturity, uint256 bal_) public view returns (uint256);

// Determines value of claim with maturity after close timestamp
function claim(uint256 maturity, uint256 bal_) public view returns (uint256);
```

### Cash Functions

`cashZero` and `cashClaim` are user facing functions that zero and claim holders call to withdraw their share of the yield token balance after the `ratio` value for their maturity timestamp is recorded.

```solidity
// Exchanges a zero balance with maturity after close timestamp for a yield token balance
function cashZero(address usr, uint256 maturity, uint256 bal_) external;

// Exchanges a claim balance with maturity after close timestamp for a yield token balance
function cashClaim(address usr, uint256 maturity, uint256 bal_) external;
```

Claim balances need to be collected until the `lastFracTimestamp` before they can be cashed out.

Sliced claim balances need to be merged back as well since they cannot be cashed out directly at the sliced timestamps.
