# [N01] `RateLimitedMinter` contract is likely missing an inheritance from the `IERC20Mintable` interface.

## Impact

The `RateLimitedMinter` contract has a `mint()` function that matches the signature of the `mint()` function declared in `IERC20Mintable`. Specifically.

```solidity
function mint(
        address to,
        uint256 amount
    ) external onlyCoreRole(role) whenNotPaused {
```
However, `RateLimitedMinter` does not explicitly inherit from IERC20Mintable by having it in the inheritance list.

[contract RateLimitedMinter
](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/rate-limits/RateLimitedMinter.sol#L17-L63)

```solidity
contract RateLimitedMinter is RateLimitedV2 {
    /// @notice the reference to token
    address public immutable token;


    /// @notice role used to access control on mint/replenishBuffer
    bytes32 public immutable role;


    /// @param _core reference to the core smart contract
    /// @param _token reference to the token to mint
    /// @param _role role used to check access control
    /// @param _maxRateLimitPerSecond maximum rate limit per second that governance can set
    /// @param _rateLimitPerSecond starting rate limit per second for minting
    /// @param _bufferCap cap on buffer size for this rate limited instance
    constructor(
        address _core,
        address _token,
        bytes32 _role,
        uint256 _maxRateLimitPerSecond,
        uint128 _rateLimitPerSecond,
        uint128 _bufferCap
    )
        CoreRef(_core)
        RateLimitedV2(_maxRateLimitPerSecond, _rateLimitPerSecond, _bufferCap)
    {
        token = _token;
        role = _role;
    }


    /// @notice Mint new tokens.
    /// Pausable and depletes the buffer, reverts if buffer is used.
    /// @param to the recipient address of the minted tokens.
    /// @param amount the amount of tokens to mint.
    function mint(
        address to,
        uint256 amount
    ) external onlyCoreRole(role) whenNotPaused {
        _depleteBuffer(amount); /// check and effects
        IERC20Mintable(token).mint(to, amount); /// interactions
    }


    /// @notice replenish the buffer.
    /// This can be used when tokens are burnt, for instance.
    /// @param amount of tokens to replenish buffer by
    function replenishBuffer(uint256 amount) external onlyCoreRole(role) {
        _replenishBuffer(amount); /// effects
    }
}
```
This suggests that the contract intends to conform to the `IERC20Mintable` interface, but simply forgot to formally inherit from it.

The impact of this is that any contract interacting with `RateLimitedMinter` and expecting it to conform to `IERC20Mintable` would work fine. However, by not formally inheriting, it could lead to confusion for developers or tools trying to analyze and validate conformance to the interface.

Additionally, if `RateLimitedMinter` ever wanted to override or change the behavior of the `mint()` function, it would break the assumptions of IERC20Mintable conformance without any compile-time checks or warnings.

## Mitigation

The `RateLimitedMinter` contract could simply formally inherit from `IERC20Mintable` to ensure the compiler enforces conformance:
```solidity
contract RateLimitedMinter is RateLimitedV2, IERC20Mintable {
  // ...
}
```

# [N02] SIGNATURE MALLEABILITY. `ecrecover` takes a signed message hash (v, r, s) and returns the Ethereum address that was used to sign it. 

## Impact

`ecrecover` takes a signed message hash (v, r, s) and returns the Ethereum address that was used to sign it. 

The issue is that the same message hash can have multiple valid signature representations that still recover to the same signer address.

This can be problematic in situations where the contract relies on the signature itself to uniquely identify or authorize something. 

In this case, the contract is using the signature to validate a delegation action - assigning delegation power from `delegatee` to another address.

The impact of signature malleability here is that an attacker could take a valid delegation signature and modify it to have a different (r, s) form, while still having it verify correctly with `ecrecover`.

This could potentially confuse logic that expects only one exact signature for a delegation. Or make duplicate delegations appear valid.

[ERC20MultiVotes.sol#L483-L501](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L483-L501)

```solidity
        address signer = ecrecover(
            keccak256(
                abi.encodePacked(
                    "\x19\x01",
                    _domainSeparatorV4(),
                    keccak256(
                        abi.encode(
                            DELEGATION_TYPEHASH,
                            delegatee,
                            nonce,
                            expiry
                        )
                    )
                )
            ),
            v,
            r,
            s
        );
```

## Mitigation

Use an alternative signature scheme like EIP-712 typed structured data signatures. These encode the data being signed in a standardized way that prevents malleability.

  * Include a nonce or unique ID value in the signed message data. Require that signatures over the same data must use the same nonce/ID. This binds the signature to prior context to prevent manipulation.

# [N03] Functions `safeCastTo232()`, `safeCastTo240()`, and `safeCastTo248()` do not follow the Solidity naming convention for internal functions.

## Impact

According to the Solidity style guide, non-external functions should have an underscore `_` prefix. However, these functions do not have the underscore prefix despite being marked `internal`.

The impact of missing the underscore prefix is primarily one of reduced code clarity and consistency:

- It makes the code less consistent to read and understand since internal functions don't all follow the same naming patterns
- Can lead to confusion between internal utility functions vs public interface methods
- Reduces adherence to established best practices for Solidity

However, there does not appear to be a direct security implication from the naming alone. The functions are properly marked `internal` which means they cannot be called externally anyway.

[safeCastTo248](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/external/solmate/SafeCastLib.sol#L8-L12), [safeCastTo240](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/external/solmate/SafeCastLib.sol#L14-L18), [safeCastTo232](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/external/solmate/SafeCastLib.sol#L20-L24)

```solidity
function safeCastTo248(uint256 x) internal pure returns (uint248 y) {
    require(x < 1 << 248);


    y = uint248(x);
}

//

function safeCastTo240(uint256 x) internal pure returns (uint240 y) {
    require(x < 1 << 240);


    y = uint240(x);
}

//

function safeCastTo232(uint256 x) internal pure returns (uint232 y) {
    require(x < 1 << 232);


    y = uint232(x);
}
```

## Mitigation

Rename the internal functions to have an underscore "_" prefix.
```solidity
function _safeCastTo232(uint256 x) internal pure returns (uint232 y) {
  // function body
}

function _safeCastTo240(uint256 x) internal pure returns (uint240 y) {
 // function body
}

function _safeCastTo248(uint256 x) internal pure returns (uint248 y) {
 // function body
}
```
Update any internal usages of these functions to call the newly renamed versions with the underscore. For example
```solidity
uint248 z = _safeCastTo248(someValue);
```

# [N04] Event parameters should be marked as `indexed` to facilitate event filtering and more efficient data retrieval.

## Impact

The root cause is simply missing best practices around event declaration in Solidity. Events are really powerful for off-chain services to track contract activity - but only if declared properly.

By not marking any event parameters as indexed, it means tools/services can only filter based on the event signature itself. They cannot selectively filter down to specific parameter values.

For example, without indexed parameters on `QuorumUpdated` - an external service could filter to get all `QuorumUpdated` events, but couldn't retrieve only events where `newQuorum` equals some value.

This limits the usefulness of the events for off-chain consumption and analytics. Dapps and services will end up having to process larger sets of log data, much of which may be irrelevant to their needs.

In terms of exploit potential - this doesn't directly introduce vulnerabilities per se. But it could contribute to some denial-of-service scenarios:

- Causing services to parse extraneous log data, inflating storage and computing needs
- Obfuscating actual critical events in a sea of unnecessary data flows
- Making the contracts overall more "noisy" to integrate with

[#L31](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L31-L31), [#L172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L172-L172), [#L194](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L194-L194), [#L30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L30-L30), [#L83](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L83-L83), [#L132](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L132-L132), [#L380](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L380-L380)

```solidity
    event QuorumUpdated(uint256 oldQuorum, uint256 newQuorum);

    event TransfersEnabled(uint256 block, uint256 timestamp);

    event ProfitManagerUpdated(uint256 timestamp, address newValue);

    event QuorumUpdated(uint256 oldQuorum, uint256 newQuorum);

    event TimelockChange(address oldTimelock, address newTimelock);

    event MaxDelegatesUpdate(uint256 oldMaxDelegates, uint256 newMaxDelegates);

    event MaxGaugesUpdate(uint256 oldMaxGauges, uint256 newMaxGauges);
```

## Mitigation
The solution is quite straightforward - declare 1-2 of the most relevant parameters in each event as indexed.

# [N05] Using `block.timestamp` or `block.number` as a proxy for time or determining time deltas can be unreliable.

## Impact

block times and numbers:

- Are not strictly constant or precise enough for many time-based calculations
- Can vary due to changes in network conditions and protocol rules
- Can shift in the event of chain reorganizations 

typical block times average around 14 seconds. But in practice there is variability.

The main impact of relying on these block values for precise timing is potential inaccuracies, surprises, or complete breaking of assumptions.

For example, in this code, the elapsed time is calculated between two block timestamps. If block intervals shorten or lengthen unexpectedly, the elapsed time would be incorrect.

Likewise, calculations based on block number over a period of time could over or underestimate based on fluctuations in block frequency. 

This could result in unintended consequences: missed expectations, improper adjudication of time-sensitive operations, etc.

[#L45](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L45-L45), [#L84](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L84-L84), [#L99](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L99-L99)

```solidity
lastBufferUsedTime = block.timestamp.safeCastTo32();
//
uint256 elapsed = block.timestamp.safeCastTo32() - lastBufferUsedTime;
//
uint32 blockTimestamp = block.timestamp.safeCastTo32();
```

## Mitigation

Rather than block parameters, I would recommend using Oracle-provided timestamps. These give much more precision without variability or blockchain-specific dependencies. 

An Oracle service can be called periodically to fetch reliable timestamps immune to the intricacies of block dynamics.