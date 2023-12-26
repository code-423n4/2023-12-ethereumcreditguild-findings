### [Low-1] Important function reverts if too many delegatees assigned

* **Description**: `ERC20MultiVotes::_decrementVotesUntilFree()` reverts with out of gas error if user has delegated more than ~300 delegates. It uses a lot of gas because of the `values()` function of OpenZeppelin's `EnumerableSet.sol` has to copy the entire storage to memory.
* **Impact**: This will cause functions like `GuildToken::applyGaugeLoss()` to revert because it calls `GuildToken::_burn()`, which first must call `_decrementVotesUntilFree()`.
* **Location**: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L437C8-L437C8
* **PoC**: `forge test --match-path ./test/unit/tokens/ERC20MultiVotes.t.sol --match-test testBurnRevertsIfTooManyDelegatees --gas-limit 30000000 -vv`
```js
    function testBurnRevertsIfTooManyDelegatees() public {

        token.mint(address(this), 100e18);
        token.setMaxDelegates(305);
        uint256[] memory delegateAmounts = new uint256[](305);

        // Delegate some amounts for the first few delegatees.
        delegateAmounts[0] = 10e18;
        delegateAmounts[1] = 20e18;
        delegateAmounts[2] = 5e18;
        
        // Make a bunch of delegate addresses and increment by their amounts
        // (For index 3 --> 305 it delegates 0 amount.
        for (uint i = 0; i < 305; i++) {    
            address delegateAddress = address(uint160(uint(keccak256(abi.encodePacked(i)))));
            uint256 amount = delegateAmounts[i]; // for i > 3 the amount defaults to 0.
            token.incrementDelegation(delegateAddress, amount); // We can increment by 0 amount.
        }

        // Calls `_burn()` under the hood which calls `_decrementVotesUntilFree()`,
        // which uses EnumerableSet `values()` function for it's array, which is gas expensive.
        // This test runs out of gas and reverts if over ~300 delegates or more.
        token.mockBurn(address(this), 50e18);
        require(token.freeVotes(address(this)) == 15e18);
    }
```
* **Recommendation**: Consider having a `MAX_DELEAGTES` immutable state variable with a value of 10-200 inside `GuildToken.sol`. 



### [Low-2] Users can call increment and decrement functions with amount 0
* **Description**: 
	* Anyone can call `CreditToken::incrementDelegation()` and pass in 0 for amount. This will still add the `delegatee` to `true` for the `delegator`.
	* Anyone can call `GuildToken::incrementGauge()` and pass in 0 for amount. This will still add the gauge to the `_userGauges` mapping.
	* Anyone can call `GuildToken::decrementGauge()` and pass in 0 for amount. This causes useless gas cost if user accidentally doesn't input an amount.
* **Impact**: This can lead to unintended consequences if protocol is expanded upon, it is never good to be able to increment/decrement values by 0 where they should always be whole numbers.
* **Locations**: 
	* https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L264-#L269 (incrementDelegation)
	* https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L317-#L342 (_incrementDelegation)
	* https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L219-#L226 (incrementGauge)
	* https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L301-#L312 (decrementGauge)
* **Recommendation**: Have a `!=0` check for the functions.



### [Low-3] Incorrect comparison operator in comments
* **Impact**: Confusion due to comments not matching with the code's logic.
* **Locations**: 
	* https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L472
	* https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L399
* **Recommendation**: To remove tokens from a user, the user's free votes/weight must be `>=` the amount to move, instead of `<`.

```diff
- /// NOTE: any "removal" of tokens from a user requires freeVotes(user) < amount.
- /// NOTE: any "removal" of tokens from a user requires userUnusedWeight < amount.
+ /// NOTE: any "removal" of tokens from a user requires freeVotes(user) >= amount.
+ /// NOTE: any "removal" of tokens from a user requires userUnusedWeight >= amount.
```
