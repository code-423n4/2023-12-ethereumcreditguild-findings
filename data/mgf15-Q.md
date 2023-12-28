# Low-1 Wrong block duration 

the constant `POLL_DURATION_BLOCKS` in `LendingTermOffboarding.sol` had value of 46523, it means that `POLL_DURATION_BLOCKS` was calculated based on a block time of 13 seconds , as the comment `// ~7 days @ 13s/block` and this is wrong block time,
the block time now is around 12s see [etherscan](https://etherscan.io/chart/blocktime):
Let's recalculate the approximate duration block time of 12s as [etherscan](https://etherscan.io/chart/blocktime):

Approximate duration in seconds = `POLL_DURATION_BLOCKS` × block time per block

while the block time new is around ~12.08 


### Substituting the values:

Approximate duration in seconds = $46523×12$ seconds/block ≈ 558276 seconds 


So, with a block time of 12 seconds, the `POLL_DURATION_BLOCKS` value of 46523 would result in an approximate poll duration of around 558,276 seconds, which is approximately 6.46 days and not 7 days.

```solidity
    uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block
```
[github](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L36)
# Low-2 `proposeOffboard` will not work as expected if the block time is not 13s

`proposeOffboard` function will not work if the block time is 13s , as of POLL_DURATION_BLOCKS is hardcoded of block time 13s look [L-1](#low-1-wrong-block-duration) this will lead to get less days and the function will revert for the users.
 
```solidity
require(
            block.number > lastPollBlock[term] + POLL_DURATION_BLOCKS,
            "LendingTermOffboarding: poll active"
        );
```
[github](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L94-L97)
# Low-3 wrong number of seconds in a year

the constant YEAR in `LendingTerm.sol` is has incorrect number of seconds .

```solidity
    uint256 public constant YEAR = 31557600;

```
[github](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L68)

Substituting the values:
Seconds in a year = $365×24×60×60$

Seconds in a year=31,536,000

# Low-4 `votingPeriod()` return wrong period based on incorrect blocktime

the function `votingPeriod()` will return wrong period based on incorrect blocktime of 13s per block as on [L1](#low-1-wrong-block-duration) the function return 2425847 and this incorrect .


```solidity
function votingPeriod() public pure override returns (uint256) {
        return 2425847; // ~1 year with 1 block every 13s
    }
```
[github](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L230-L232)

If the block time is 12 seconds, and the votingPeriod function is returning a constant value of 2425847, you can recalculate the approximate duration in seconds using the updated block time:

Approximate duration in seconds=voting period blocks×block time per block

Substituting the values:

Approximate duration in seconds = $2425847×12 seconds/block≈29110164$ seconds

So, with a block time of 12 seconds, the votingPeriod value of 2425847 would result in an approximate voting period duration of around 29,110,164 seconds, which is still approximately 337 days or about 11 months.

If the goal is to have a voting period lasting about 1 year with a block time of 12 seconds, you might need to adjust the constant value. One way to approach this is to calculate the desired number of blocks for a 1-year duration with a 12-second block time:

Desired voting period blocks = $\frac{desired duration in seconds} {block time per block}$

For a 1-year duration:

### Desired voting period blocks = $\frac{365×24×60×60}{12}≈2628000$

# Low-5 hardcoded `POLL_DURATION_BLOCKS` value can lead to inaccurate distribution

use the constant `POLL_DURATION_BLOCKS` value of 13s block time can lead to inaccurate distribution , that because blocktime is changing over time .

![img](https://i.imgur.com/FWyoMjd.png)

ref [ethereum_average_block_time](https://ycharts.com/indicators/ethereum_average_block_time)


# N-1 typo 

    /// Quorum managment. //@audit management

[link](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L53)

    /// @notice Allow or disallow a given implemenation //@audit implementation 
[link](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L91)


# N-2 no need to emit block.timestamp

some event emit block.timestamp no need to do this since the timestamp is already on the block .