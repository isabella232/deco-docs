# Build an integration

Deco's primary integration point with all yield protocols is the yield token balance. This approach allows Deco to be integrated with ease. We have designed Deco to allow developers to customize a few key interfaces to allow Deco to meet the needs of yield protocols and bring fixed-rates as a feature to your own ecosystems and use cases.

The following sections outline various functions within the `deco-base` repo that have to be customized for each yield protocol before deployment to handle all the yield tokens issued by it. For ex: One customization for Compound(or Aave) will be sufficient to deploy Deco instances that can handle all cTokens( or aTokens).

## Lock, Unlock

`lock` and `unlock` are internal functions which implement the functionality to transfer a yield token balance between a user and the Deco instance. They are setup to handle ERC20 yield token balance transfers by default.

The `tokenDecimals` value is read from the Yield token contract and set within the constructor to the amount of decimals used by it.

Both `lock` and `unlock` are designed to use this value and issue Zero and Claim balances with 18 decimals irrespective of the number of decimals used by the yield token.

Since the decimals information is known before deployment, the `tokenDecimals` value can be hard-coded and the `if-else` conditions can be removed from `lock` and `unlock` to save gas costs for users of the instance.

## Governance

Each Deco instance can pick a governance mechanism based on its own needs. The correct address when governance is enabled needs to be set within the constructor at deployment-time. 

Optionally, `updateGov` function can be enabled to give an ability to update the address after deployment.

## Fraction value capture

Deco instances have to implement either `snapshot` or `insert` functions. We recomment developers to also implement `insert` as a backstop even when `snapshot` is implemented.

Developers have to apply appropriate transformations within `snapshot` to ensure the fraction value captured is reflective of the actual amount received when yield tokens are redeemed by users later at their yield protocol at various redemption sizes including fees.

Within `insert`, governance is allowed to calculate the fraction value externally and directly insert it at a timestamo.
It is up to developers to allow governance to overwrite existing `frac` values, insert values at future timestamps etc. Both are dangerous practives and should typically not be allowed.

Sometimes, the implementation of both `snapshot` and `insert` can be skipped if all outstanding issuances at various maturity timestamps on a Deco instance are meant to be settled just once which can be done by repurposing `close`.

## Close

`close` can be a public function when the conditions it can be triggered under are read from yield protocol or elsewhere on-chain. Ex: Deco instances attached to the Maker protocol can setup close to execute when a change is detected in the emergency shutdown flag `live`.

We recommend developers to also give governance access to trigger Close to ensure a broad range of protocol and market instabilities can be factored in and the Deco instance shutdown to protect both users and the yield protocols.

## After-close Valuation

`calculate` function needs to be implemented with an on-chain valuation calculation, or control should be handed over to governance to valuate assets off-chain and update deposit split `ratios` between zero and claim balances for all future maturity timestamps.

Oracles are a great fit here and can be used to perform the valuation calculations off-chain and insert them after close.

Simplest on-chain valuation calculation that can be implemented is to assign a ratio of `1.00` for all future maturities to give Zero holders the entire yield token balance and leave Claim holders with nothing. Developers need to carefully consider the incentives this creates for Zero holders to cause instabilties and receive a full and early redemption of their zero. Lower ratios could also be hard-coded if appropriate.

We hope you will look at the `deco-base` repo and consider building an integration to solve a problem!
