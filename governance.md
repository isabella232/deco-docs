# Governance

Operating a Deco instance successfully for a yield token requires the following external data to be input into it on a regular basis:

* Fraction value snapshots
* Monitoring the safety and solvency of a yield protocol to determine whether the attached Deco instance should be closed proactively to protect users.
* Adding zero and claim valuation data to process withdrawals after close.

Governance mechanisms can be set up to input this data whenever Deco instances are not able to capture it themselves directly from the yield protocol. Even in scenarios where the data can be captured in a trustless manner, it might only be available on-chain for a very short duration which could easily result in a failure to capture it within Deco at the right time. Governance mechanisms serve as a crucial backup mechanism who can be trusted to input this data ex post facto in such scenarios and prevent inevitable losses for users.

The following functions can be restricted and setup with the `onlyGov` modifier to hand over control to the governance mechanism.

`updateGov` function is restricted to allow governance mechanism to update to another address. 

`insert` function when implemented has to be restricted to governance because `snapshot` is its equivalent public function. Governance can insert a fraction value at any precise timestamps to allow users to settle exactly at maturity.

`close` can be setup as a restricted function operated by governance to give the operator more latitude on decisions to close the Deco instance.

`calculate` has to be restricted and executed by governance if implementing an on-chain valuation function is impossible or cost-prohibitive due to gas costs.

It is important to keep in mind that all Deco deployments are independent which gives each one the ability to select where it wants to be on the trustless spectrum and select the trusted governance body. It is possible for multiple Deco instances to be deployed for the same yield protocol and allow each one to operate at different parts of the trust spectrum to serve differing market participants’ needs.
