# QA report

## Table Of Contents

- [Gas report](#gas-report)
  - [Table Of Contents](#table-of-contents)
  - [No revert when `userFreeVotes >= votes`](#no-revert-when-userfreevotes--votes)

## No revert when `userFreeVotes >= votes`

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L431

It's good practice to revert a token transfer transaction if it hit some condition (in this case `userFreeVotes >= votes`). It might be more suitable when we want a clear and explicit way to indicate an exceptional condition that shouldn't be continued because, `revert` will provide a custom error message that will be visible to the caller. This can be helpful for debugging and providing meaningful feedback.

### Recommended mitigation step

```solidity
// /src/tokens/ERC20MultiVotes.sol

431:        if (userFreeVotes >= votes) revert("Example: already free");
```