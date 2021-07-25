# Build an integration

Deco's primary integration point with all yield protocols is the yield token balance which allows it to be integrated with ease. We have designed Deco to allow developers to customize various interfaces to meet the needs of all yield protocols and bring fixed-rates as a feature to your own ecosystems.

The following sections outline various functions within the `deco-base` repo that have to be customized for each yield protocol before deployment for all the yield tokens under it.

## Lock, Unlock

The `tokenDecimals` value is read from the Yield token contract and set within the constructor to the amount of decimals used by it.

Both `lock` and `unlock` are designed to use this value and issue Zero and Claim balances with 18 decimals irrespective of the number of decimals used by the yield token.

Since the decimals information is known before deployment, the `tokenDecimals` value can be hard-coded and the `if-else` conditions can be removed from `lock` and `unlock` to save gas costs for users of this Deco instance.

## Governance

Each Deco instance can pick a governance mechanism based on its own needs. The correct address when governance is enabled needs to be set within the constructor at deployment-time, or updated with the `updateGov` function after deployment.

## Fraction value capture

Deco instances have to implement either `snapshot` or `insert` functions. We recomment developers to also implement `insert` as a backstop when `snapshot` is implemented.

Developers have to apply appropriate transformations within `snapshot` to ensure the fraction value captured is reflective of the actual amount received when yield tokens are redeemed at their yield protocol at all redemption sizes including fees.

Within `insert`, it is up to developers to allow governance to overwrite existing `frac` values, insert values at future timestamps etc. Both are dangerous practives and should typically not be allowed.

Sometimes, the implementation of both `snapshot` and `insert` can be skipped if all issuances of the Deco instance are meant to be settled once which can be done with `close`.

## Close

`close` can be a public function when the conditions it can be triggered under are read from yield protocol or elsewhere on-chain.

We recommend developers to also give governance access to trigger Close to ensure a broad range of instabilities can be considered and the Deco instance shutdown to protect both users and the yield protocols.

## After-close Valuation

`calculate` function needs to be implemented with an on-chain valuation, or control should be handed over to governance to update ratios for all future maturity timestamps.

Oracles are a great fit here and can be used to perform the valuation calculations off-chain and insert them after close.

Simplest on-chain valuation calculation that can be implemented is to assign a ratio of `1` for all future maturities to give Zero holders the entire yield token balance. Developers need to carefully consider the incentives this creates for Zero holders to cause instabilties and receive a full and early redemption. Lower ratios could also be hard-coded if appropriate.
