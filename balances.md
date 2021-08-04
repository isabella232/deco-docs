# Balances

Yield tokens of the same class(cTokens, aTokens, yTokens, etc.) share a Deco protocol implementation customized for their yield protocol, and each yield token(cDai, cUSDC, aLINK, yETH, etc.) is setup with its own deployed Deco protocol instance.

These Deco instances are able to handle zero and claim token balances of all maturity timestamps for the yield token from this single deployed instance.

`ZERO` and `CLAIM` token balances use `18` decimals.

All issued zero and claim balances in a Deco instance are separated based on their respective `class` values under `zBal` and `cBal` nested mappings.

```solidity
// user address => zero class => zero balance [wad: 18 decimal fixed point number]
mapping(address => mapping(bytes32 => uint256)) public zBal;

// user address => claim class => claim balance [wad: 18 decimal fixed point number]
mapping(address => mapping(bytes32 => uint256)) public cBal;
```

### Token Notation

Standard notation for all zero and claim balance symbols will be a hyphenation of `ZERO` and `CLAIM` with the yield token symbol . For example, zero and claim tokens issued for a Deco instance setup for the yield token CHAI would be denoted with `ZERO-CHAI` and `CLAIM-CHAI`.

## Balance Class

`class` mechanism allows zero and claim balance amounts that share common characteristics based on their `issuance` and `maturity` timestamps to be fungible when transferred among addresses.

### Zero

Zero balance amounts with the same `maturity` timestamp are fungible with each other since each unit can be redeemed after this timestamp for the same amount of yield token balance.

Zero `class` value of a balance is derived as:

```solidity
// calculate zero class with maturity timestamp
bytes32 class_ = keccak256(abi.encodePacked(maturity));
```

### Claim

Claim balance amounts with the same `issuance` timestamp and `maturity` timestamp are fungible since each unit collects the same amount of yield earned between these timestamps.

Claim `class` value of a balance is derived as:

```solidity
// calculate claim class with both issuance and maturity timestamps
bytes32 class_ = keccak256(abi.encodePacked(issuance, maturity));
```

## Balance Transfers

`moveZero` and `moveClaim` functions allow users to transfer their internal balances tracked in `zBal` and `cBal` mappings.

```solidity
// Transfers user's zero balance
function moveZero(address src, address dst, bytes32 class_, uint256 bal_) external;

// Transfers user's claim balance
function moveClaim(address src, address dst, bytes32 class_, uint256 bal_) external;
```

## Balance Adapters

Users have the choice to convert their internal balances at any time into either ERC20 or ERC721 standard token balances based on their needs.

The following balance adapter smart contracts facilitate this conversion for users.

|       | ERC20             | ERC721             |
|-------|-------------------|--------------------|
| ZERO  | ZeroAdapterERC20  | ZeroAdapterERC721  |
| CLAIM | ClaimAdapterERC20 | ClaimAdapterERC721 |

Each one of these four smart contracts implement the `exit` and `join` functions:

* Exit allows a user to convert an internal balance to the standard token balance offered by the adapter.
* Join allows a user to convert the standard token balance back to an internal balance.

```solidity
// contract: ZeroAdapterERC20.sol
// Converts internal zero balance to its ERC20 token balance
function exit(address src, address dst, bytes32 class_, uint256 zbal_) external;
// Converts ERC20 token balance back to internal zero balance
function join(address src, address dst, bytes32 class_, uint256 zbal_) external;

// contract: ClaimAdapterERC20.sol
// Converts internal claim balance to its ERC20 token balance
function exit(address src, address dst, bytes32 class_, uint256 cbal_) external;
// Converts ERC20 token balance back to internal claim balance
function join(address src, address dst, bytes32 class_, uint256 cbal_) external;

// contract: ZeroAdapterERC721.sol
// Converts internal zero balance to an NFT
function exit(address src, address dst, bytes32 class_, uint256 zbal_) external;
// Converts NFT back to internal zero balance
function join(address src, address dst, uint256 tokenId_) external;

// contract: ClaimAdapterERC721.sol
// Converts internal claim balance to an NFT
function exit(address src, address dst, bytes32 class_, uint256 cbal_) external;
// Converts NFT back to internal claim balance
function join(address src, address dst, uint256 tokenId_) external;
```

Adapters are only used to represent balances and allow users to transfer them to other addresses in a standard fashion. Balances have to be converted back to their internal form to interact with core Deco functionality like settlement after maturity.

Internal balance is held in custody by these contracts when issued ERC20 and ERC721 token balances are in circulation.

At this point, users would have to use the internal `approve` function to set an approval for the balance adapter contract addresses before they can be used to convert internal balances for them.

### ERC20

Before an internal balance can be converted, an ERC20 token needs to be deployed. Anyone can call `deployToken` to deploy the token contract for a class value. `tokens` mapping will store the deployed ERC20 token contract address for its class and it will remain unique thereafter.

```solidity
// contract: ZeroAdapterERC20.sol
// Deploys ERC20 token for a zero class
function deployToken(uint256 maturity) public returns (address);

// contract: ClaimAdapterERC20.sol
// Deploys ERC20 token for a claim class
function deployToken(uint256 issuance, uint256 maturity) public returns (address);
```

It is usually the Deco instance operator's responsibility to deploy ERC20 token contracts from the balance adapter on behalf of all users for standard maturity timestamps in the market that have a lot of user activity like the end of the month, quarter, year, etc. After deployment, everyone is free to use the deployed token contract addresses to convert their internal balances.

Zero ERC20 token name is set to a string that contains the token symbol and the maturity timestamp in unix time separated by a space.

Claim ERC20 token name contains the token symbol, issuance timestamp, and maturity timestamps in unix time separated by spaces.

ERC20 token symbols match their token symbols and do not contain the timestamp information like the name.

ERC20 token decimals are always set to `18` to match the internal balances.

### ERC721

Unlike ERC20 tokens, no new token contracts need to be deployed by users from the ERC721 adapters for zero and Claim balances. All NFTs representing the converted balances for any class are issued directly from `ZeroAdapterERC20` and `ClaimAdapterERC20` contracts.

Each NFT records its class and balance amount metadata under the `class` and `amount` mappings.

Token IDs are assigned incrementally and are not re-used once the balance represented by an NFT is converted back again to an internal balance.

Metadata URI is not set for tokens since the required information is stored within both the contracts.

## Approvals

Users can approve other addresses to manage all their internal balances by setting the approval boolean flag to `true`, or revoke previous approval by setting it to `false`, with the `approve` function.

```solidity
// balance address => approved address => approval status
mapping(address => mapping(address => bool)) public approvals;
```

The `approved(address usr)` function modifier is applied to all user facing functions within Deco, including Core, and Balance Adapter contracts.

All Deco functions explicitly specify the `src` address(instead of only operating on the `msg.sender` balance) as the source of the internal balance to act upon, and the `dst`  address in all function inputs. This allows owners to set an approval for their own DSProxy or similar smart wallet contracts and execute functions in a batch without having to transfer ownership of the balance to the smart wallet and back after the batch transaction execution.
