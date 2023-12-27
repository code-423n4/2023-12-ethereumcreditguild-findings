# [L-01] Invalid `POLL_DURATION_BLOCKS` constant makes polls end sooner than intended

## Impact
As offboard proposals will end sooner than intended but the project is designed in many places like [here](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L25) or [here](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L189) to work with $7$ days periods, it will be returning an invalid status of the given proposals.

## Proof of Concept
Since the migration to POS, Ethereum now creates blocks at a fixed rate of 12 seconds, see [here](https://etherscan.io/chart/blocktime) for a tangible proof. However, the project assumes a wrong generation rate, leading to proposals ending, roughly, one hour sooner than intended.

[**LendingTermOffboarding, line 36**](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L33C1-L36C81)

```solidity
    /// @notice maximum age of polls for them to be considered valid.
    /// This offboarding mechanism is meant to be used in a reactive fashion, and
    /// polls should not stay open for a long time.
    uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block
```

Although it seems like no harm, and because the front-end is OOS, I can't show as a POC an offboard proposal being in a voting period where users are available to vote, but their transactions will revert as on-chain, the proposal has ended, leading to a loss of gas from an user POV.

## Recommended Mitigation Steps

I would personally use as a time measure `block.timestamp`, but that would make your team re-design the whole contract. Because of that, to minimize the number of changes, this is the solution I recommend:

1. Make `POLL_DURATION_BLOCKS` non-constant and initialize it correctly

```diff
-   uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block
+   uint256 public pollDurationBlocks = 50400; // ~7 days @ 12s/block
```

2. Use a priviledged function to update it if necessary

```solidity
    event pollDurationBlocksUpdated(uint256 prev, uint256 new);

    function setPollDurationBlocks(uint256 newPollDurationBlocks) external onlyCoreRole(CoreRoles.GOVERNOR) {
        emit pollDurationBlocksUpdated(pollDurationBlocks, newPollDurationBlocks);

        pollDurationBlocks = newPollDurationBlocks;
    }
```