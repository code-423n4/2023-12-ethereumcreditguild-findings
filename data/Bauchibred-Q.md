# QA Report

## Q-01 Protocol can't correctly relay if the `votingPeriod` is over to users

### Proof Of Concept

Take a look at [votingPeriod()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L226-L232)

```solidity
   function votingPeriod() public pure override returns (uint256) {
        return 2425847;  // ~1 year with 1 block every 13s
    }
```

As seen, this function is used to return the voting period, evidently this function returns `2425847`, issue is that this should be 1 year but protocol makes the assumption that **1 block is mined every 13s**, now this makes protocol off over a month on the ethereum mainnet, this is cause `2425847` amounts to around 336 days which is 1 month of the intended value.

Now to worsen this, is the fact that protocol is actually going to be deployed not only on the Ethereum mainnet but even Arbitrum type L2s as clearly stated [here](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/README.md?plain=1#L123), what this now leads to is having a way off number for the voting period depending on what chain this gets deployed, using Arbitrum as a case study, from [their official docs](https://docs.arbitrum.io/for-devs/concepts/differences-between-arbitrum-ethereum/block-numbers-and-time#block-numbers-arbitrum-vs-ethereum), one can see that a single Ethereum block includes multiple Arbitrum blocks within it which leads to this value been shorter than necessary in this case, and if takes it shorter on a different Arbitrum like chain then the same case would be made.

### Impact

Protocol can't rightly relay if the `votingPeriod` is over to users, i.e after the action is ready in the timelock, probably limiting users engagements in voting since they might assume voting's over when it's not.

### Recommended Mitigation Steps

Do not hardcode the `votingPeriod` value for all chains and also it shouldn't be `2425847` for the Ethereum mainnet as this is one month off than the original value.


## Q-02 While creating a new term, an interest rate of 100% should be acceptable


### Proof of Concept

Take a look at [createTerm()]()

```solidity
    function createTerm(
        address implementation,
        LendingTerm.LendingTermParams calldata params
    ) external returns (address) {
        require(
            implementations[implementation],
            "LendingTermOnboarding: invalid implementation"
        );
        // must be an ERC20 (maybe, at least it prevents dumb input mistakes)
        (bool success, bytes memory returned) = params.collateralToken.call(
            abi.encodeWithSelector(IERC20.totalSupply.selector)
        );
        require(
            success && returned.length == 32,
            "LendingTermOnboarding: invalid collateralToken"
        );

        require(
            params.maxDebtPerCollateralToken != 0, // must be able to mint non-zero debt
            "LendingTermOnboarding: invalid maxDebtPerCollateralToken"
        );

        require(
            //@audit
            params.interestRate < 1e18, // interest rate [0, 100[% APR
            "LendingTermOnboarding: invalid interestRate"
        );

        require(
            // 31557601 comes from the constant LendingTerm.YEAR() + 1
            params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]
            "LendingTermOnboarding: invalid maxDelayBetweenPartialRepay"
        );

        require(
            params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%
            "LendingTermOnboarding: invalid minPartialRepayPercent"
        );

        require(
            //@audit
            params.openingFee <= 0.1e18, // open fee expected [0, 10]%
            "LendingTermOnboarding: invalid openingFee"
        );

        require(
            params.hardCap != 0, // non-zero hardcap
            "LendingTermOnboarding: invalid hardCap"
        );

        address term = Clones.clone(implementation);
        LendingTerm(term).initialize(
            address(core()),
            LendingTerm.LendingTermReferences({
                profitManager: profitManager,
                guildToken: guildToken,
                auctionHouse: auctionHouse,
                creditMinter: creditMinter,
                creditToken: creditToken
            }),
            params
        );
        created[term] = block.timestamp;
        emit TermCreated(block.timestamp, implementation, term, params);
        return term;
    }
```

As seen with the help of the @audit tags, unlike with the fees where the limits are acceptable, i.e 0 and 10%, in regards to the interest rate it **must be** less than 100% which is an unintended implementation as commented by the docs.


### Impact

Broken logic/ Wrong Docs
### Recommended Mitigation Steps
Change this:
```solidity
        require(
            //@audit
            params.interestRate < 1e18, // interest rate [0, 100[% APR
            "LendingTermOnboarding: invalid interestRate"
        );
```
to:
```solidity
        require(
            //@audit
            params.interestRate <+ 1e18, // interest rate [0, 100[% APR
            "LendingTermOnboarding: invalid interestRate"
        );
```
## Q-03 Protcol's  `canExceed...`  implemented functionalities should have a somewhat second cap limit attached to them


### Proof of Concept
Using this search command: [https://github.com/search?q=repo%3Acode-423n4%2F2023-12-ethereumcreditguild%20canexceed&type=code](https://github.com/search?q=repo%3Acode-423n4%2F2023-12-ethereumcreditguild%20canexceed&type=code), i.e checking for instances of `canExceed...` we can see that protocol uses this to give out special treatment to some addresses in different logic, issue with this is that in the case where the list grows too much, be it that of delegates in the `ERC20MultiVotes.sol` contract or gueages in the `ERC20Guages.sol` contract users then functionality would be massively affected
For example in regards to the multivotes system The identified issue arises when accounts, which are allowed to surpass the `canContractExceedMaxDelegates` global limit, accumulate an excessively large number of delegates. These accounts are not restricted by the `maxDelegates` variable, leading to potential problems. Specifically, when new delegates are added, there's no check to prevent an out-of-gas (OOG) exception during iterations over an extensive delegate list. Although there is a provision to remove delegates individually, it requires that the delegates have no allocated votes. In scenarios where a delegator has numerous delegates, each with allocated votes, both the burn and transfer functions for the delegator are at risk of a denial-of-service (DoS). This is because the operations attempt to iterate through the entire delegate list to free votes, potentially triggering an OOG exception, thereby causing the burn and transfer functions to consistently revert.




### Impact

Multiple functionalities are at risk of falling into a DoS if the list grows too much and the delegates don't have free votes to allow the delegator to remove their delegates individually.
### Recommended Mitigation Steps
Maybe an idea to fix this would be to set a second cap limit on the number of delegates that a delegator that is allowed to skip the first limit can delegate their votes to.
## Q-04 Setters/Updaters should always have equality checkes


### Proof of Concept

Take a look at [updateTimelock()]()

```solidity
    function updateTimelock(
        address newTimelock
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        _updateTimelock(newTimelock);
    }

    function _updateTimelock(address newTimelock) private {
        emit TimelockChange(timelock, newTimelock);
        timelock = newTimelock;
    }
```

As seen the function is used to update the timelock the veto governor can cancel from, but there are no checks to ensure that the ew value that's been set is not equal to the previous one.

NB: This is the same for all setter/updater instances
### Impact
Low
### Recommended Mitigation Steps
Implement checks that the new value that's to be set is not the same as what was previously there.

##  Q-05 Posssible inaccurate event emission in `_replenishBuffer()`


### Proof of Concept

Take a look at `_replenishBuffer()`

```solidity
function _replenishBuffer(uint256 amount) internal {
    uint256 newBuffer = buffer();

    uint256 _bufferCap = bufferCap; // gas optimization

    // Logic to calculate the actual amount replenished
    if (newBuffer == _bufferCap) {
        return;
    }

    uint32 blockTimestamp = block.timestamp.safeCastTo32();
    uint224 newBufferStored = Math.min(newBuffer + amount, _bufferCap).safeCastTo224();

    lastBufferUsedTime = blockTimestamp;
    bufferStored = newBufferStored;

    // @audit Inaccurate event emission
    emit BufferReplenished(amount, bufferStored);
}
```

The event `BufferReplenished(amount, bufferStored)` emits the `amount` parameter as the amount replenished. However, the actual replenished amount may be less than `amount` if adding `amount` to the current buffer exceeds the buffer cap. The function correctly calculates `newBufferStored` as the minimum of `newBuffer + amount` and `_bufferCap`, but the event does not reflect this calculation.
### Impact

The `_replenishBuffer` function in the `RateLimitedV2` contract emits an event that inaccurately represents the amount of buffer replenished. This inaccuracy arises because the event emission does not account for cases where the actual replenished amount is less than the requested amount. This could lead to misleading information being recorded on the blockchain, causing confusion and potential misinterpretation of the contract's state and actions.
### Recommended Mitigation Steps

Modify the event emission to accurately reflect the actual amount replenished. This can be done by calculating the difference between `newBufferStored` and the buffer size before replenishment.

## Q-06  Return value of `mint` and `burn` are not checked could lead to


### Proof of Concept
Using these search commands:
- [https://github.com/search?q=repo%3Acode-423n4%2F2023-12-ethereumcreditguild+.mint%28&type=code](https://github.com/search?q=repo%3Acode-423n4%2F2023-12-ethereumcreditguild+.mint%28&type=code)
&
- [https://github.com/search?q=repo%3Acode-423n4%2F2023-12-ethereumcreditguild+.burn%28&type=code](https://github.com/search?q=repo%3Acode-423n4%2F2023-12-ethereumcreditguild+.burn%28&type=code)

We can see that there are multiple instances of where the protcol mints to or burns from an address, but the return values of most of these operations are not checked, for e.g in ERC20RebaseDistributor.sol

### Impact

Since the return value of `mint` and `burn` are not checked could lead to instances where the minting is not successful but protocol assumes they are, probably leading to users missing out on their tokens or protocol leaking value in an attempt to burn tokens that doesn't get successful

### Recommended Mitigation Steps
Check the return value of these executions
## Q-07  `nonRebasingSupply()` should be made more efficient

### Proof of Concept

The `nonRebasingSupply` function in a given smart contract is designed to calculate the non-rebasing supply of tokens by subtracting the `rebasingSupply` from the `totalSupply`. However, there is a potential issue in the comparison logic used to handle rounding errors and prevent underflows.


```solidity
function nonRebasingSupply() external view virtual returns (uint256) {
    uint256 _totalSupply = totalSupply();
    uint256 _rebasingSupply = rebasingSupply();

    // Comparison to handle potential underflow due to rounding errors
    if (_rebasingSupply > _totalSupply) {
        return 0;
    } else {
        return _totalSupply - _rebasingSupply;
    }
}
```


Currently, the function checks if `_rebasingSupply` is greater than `_totalSupply` to prevent underflow when subtracting. However, there is an audit note suggesting that this comparison should use a ">=" check. This is based on the reasoning that if `_rebasingSupply` equals `_totalSupply`, the subtraction would result in zero anyway, making the entire operation redundant.

### Impact

Failing to implement the suggested ">=" check could lead to unnecessary subtraction operations when `_rebasingSupply` and `_totalSupply` are equal. While this won't result in incorrect calculations or critical issues, optimizing the check can improve the function's efficiency and clarity, ensuring that it only performs subtraction when necessary.

### Recommended Mitigation Steps

Update the comparison in the `if` statement to use ">=" instead of ">". This change will ensure that the function directly returns `0` when `_rebasingSupply` is equal to or greater than `_totalSupply`, thus avoiding an unnecessary subtraction operation in these cases. This adjustment not only aligns with logical expectations but also optimizes the function for cases where both values are equal.
