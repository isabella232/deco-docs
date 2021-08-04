# Deco Protocol

> Fixed-rate converter for any yield token.

Yield token holders of most yield protocols usually receive a market driven floating-yield. Deco protocol can be integrated with any yield token to tokenize and separate its yield volatility by decomposing the yield token into two new tokens,

* **Zero**,  Yield token holders exchange their balance for a Zero balance to receive a fixed-yield for a certain time period.
* **Claim**,  Allows traders to take on the future rate volatility risk and earn only the yield by purchasing a Claim balance at a fixed cost upfront.

Example of Zero and Claim issuance with Dai as the base asset and CHAI as the yield token/protocol,

![Overview](_media/architecture.png ':size=600')

The next sections will equip you with an understanding of the core features of Deco protocol,

* [Core Lifecycle](/lifecycle.md) covers steps like issuance and redemption of Zero and Claim tokens.
* [Liquidity](/liquidity.md) explains features that enhance the liquidity profile of Deco assets to allow users greater flexibility without compromise. For ex: Deco token balances can be ERC20 or ERC721, issuance can target any date in the past, and all yield earned by Claims can be unlocked as soon as it is booked instead of once at maturity.
* [Safety](/safety.md) showcases the various user level and protocol level safety mechanisms that have been incorporated so that assets can be issued safely, and with very long maturities, instead of being restricted, and short terms like 3-6 months, and still avoid risk. Yield protocol governance can shut down all outstanding maturities and allow them to be unlocked from Deco in one step if the need arises.
* [Governance](/governance.md) outlines how Deco allows customization and can bring everything it offers under the control of your own governance/DAO and remove any external points of control.

With an understanding of the core protocol, it is time to look at our [integrations guide](build-integration.md). Deco protocol offers a base repo that is not only light-weight and safe, but also designed for flexible and quick integrations with a wide variety of yield tokens and protocols. You can see it in action in the integrations we have discovered so far and we hope you consider building one to seamlessly bring fixed-rates as a feature to your protocol and governance.

Send us a note at `info@deco.money` with your questions regarding integrations with Deco and we will help you out.
