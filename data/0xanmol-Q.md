## [L-1]  `MinStake` is constant in `SurplusGuildMinter` and it may not be feasible for all markets

### Code Line

https://github.com/volt-protocol/ethereum-credit-guild/blob/4d33abf95fee69391af0652e3cbe5e0cffa25f9f/src/loan/SurplusGuildMinter.sol#L26C46-L26C46

### Impact

In the `SurplusGuildMinter` contract, `minStake` is the minimum amount of credit that users need to stake in order to increase the gauge weight. Currently, this variable is set as a constant, with a value of 1e18, meaning it cannot be changed and always represents 1e18 GUSDC. This works fine when dealing with the USDC:GUSDC market, as GUSDC is assumed to be pegged to USDC. However, a problem arises when there are multiple markets.

### Proof of Concept

Let's consider the scenario where the team decides to add a new market for WETH:GWETH, where GWETH is pegged to WETH. If the same `SurplusGuildMinter` contract is deployed with changes to the constructor variable (which will happen because the contract is immutable), the 1e18 `minStake` amount will not be feasible for GWETH. 1e18 represents 1 GWETH, which is a significant amount to be set as the minimum.

### Recommendation

To address this issue, it is recommended to make `minStake` an immutable variable and set the feasible minimum stake amount when deploying the contract.

## [L-2] `NotifyPnL` is not checking if the loss == _surplusBuffer

### Code Line

https://github.com/volt-protocol/ethereum-credit-guild/blob/4d33abf95fee69391af0652e3cbe5e0cffa25f9f/src/governance/ProfitManager.sol#L315

### Details

When the gauge receives notification of a loss, it verifies whether the loss is smaller than the _surplusBuffer and if it can be covered by the _surplusBuffer amount. However, if the `_surplusBuffer == loss`, instead of covering the entire loss, the code mistakenly proceeds to the else statement where it attempts to decrease the `creditMultiplier`. This is unnecessary and inefficient.

### Recommendation

Instead of solely checking if the loss is < _surplusBuffer, it should also verify if it is equal to _surplusBuffer by using loss ≤ _surplusBuffer.

```diff
- if (loss < _surplusBuffer){
+ if (loss <= _surplusBuffer){

                // deplete the surplus buffer
                surplusBuffer = _surplusBuffer - loss;
                emit SurplusBufferUpdate(
                    block.timestamp,
                    _surplusBuffer - loss
                );
                CreditToken(_credit).burn(loss);
            } else {
                // empty the surplus buffer
                loss -= _surplusBuffer;
                surplusBuffer = 0;
                CreditToken(_credit).burn(_surplusBuffer);
                emit SurplusBufferUpdate(block.timestamp, 0);

                // update the CREDIT multiplier
                uint256 creditTotalSupply = CreditToken(_credit).totalSupply();
                uint256 newCreditMultiplier = (creditMultiplier *
                    (creditTotalSupply - loss)) / creditTotalSupply;
                creditMultiplier = newCreditMultiplier;
```
## [L-3]  `partialRepayDelayPassed` should return true if the term is offboarded.

### Code Line

https://github.com/volt-protocol/ethereum-credit-guild/blob/4d33abf95fee69391af0652e3cbe5e0cffa25f9f/src/loan/LendingTerm.sol#L237

### Details

The variable `LendingTerm:partialRepayDelayPassed` is used to check whether the partial repayment deadline for a loan has passed or not. It returns "true" if the deadline has passed, and "false" if it hasn't.

If the term is offboarded, it becomes callable and doesn't need the partial repayment delay to pass. In this case, the variable can return "true" even if the repayment period hasn't passed yet. This is done to prevent any potential errors when the protocol interacts with this term.

```diff
function partialRepayDelayPassed(
        bytes32 loanId
    ) public view returns (bool) {
        // if no periodic partial repays are expected, always return false
        if (params.maxDelayBetweenPartialRepay == 0) return false;

        // if loan doesn't exist, return false
        if (loans[loanId].borrowTime == 0) return false;

        // if loan is closed, return false
        if (loans[loanId].closeTime != 0) return false;

+ if(GuildToken(refs.guildToken).isDeprecatedGauge(address(this)) return true;
        // return true if delay is passed
        return
            lastPartialRepay[loanId] <
            block.timestamp - params.maxDelayBetweenPartialRepay;
    }
```
## [L-4] Allowing partial repayment and add collateral after offboarding can cause users to lose funds.

### Code Line

https://github.com/volt-protocol/ethereum-credit-guild/blob/4d33abf95fee69391af0652e3cbe5e0cffa25f9f/src/loan/LendingTerm.sol#L490

### Details

If the term is offboarded, all the loans within that term need to be called. This action is typically carried out by a centralized keeper.
However, if the user or any other interacting protocol is not aware of the offboarding, they can reduce their debt by calling the partialRepay function before the keeper initiates the loan calls. By doing so, their debt will decrease while the underlying collateral remains the same.

These types of loans are highly attractive to auction bidders, who can quickly place bids on them before users realize and offer their credit at a lower price to settle debt and seize collateral.
To avoid such situations altogether, it is advisable to prohibit repayments once the term is deprecated.

Along with partial repayments, the option to add collateral should also be disallowed after the term is deprecated. Allowing users to add collateral after the term is deprecated would unnecessarily expose their additional collateral to risk.

### POC
effect of partial repayment on position with 20% interest and 0 openingFee

```solidity
  function test_seizeBorrowerCollateralForLessDebt() public {
        //Increment weight
        guild.mint(address(this), 200e18);
        guild.incrementGauge(address(term), 100e18);
        //Borrow Credits
        collateral.mint(address(this), 100e18);
        collateral.approve(address(term), 100e18);
        bytes32 loanId = term.borrow(1000e18, 100e18);
        assertEq(term.getLoanDebt(loanId), 1000e18); //without any interest

        //Offboard the term after 10 days
        skip(10 days);
        guild.removeGauge(address(term));
        //user try to partial repay
        uint loanAmount = term.getLoanDebt(loanId);

        LendingTerm.Loan memory loanDetailsBefore = term.getLoan(loanId);
        console.log("Total debt after 10 days of taking loan", loanAmount);
        uint payHalf = loanAmount / 2;
        console.log("rePaying half ", payHalf);
        console.log(
            "collateral before partial repay",
            loanDetailsBefore.collateralAmount
        );

        console.log("-----------------------------------------");
        credit.approve(address(term), payHalf);
        //@audit-issue don't allow user to partial repay
        term.partialRepay(loanId, payHalf);

        uint debtAfterRepay = term.getLoanDebt(loanId);
        console.log("Total debt after partial repay", debtAfterRepay);
        LendingTerm.Loan memory loanDetailsAfter = term.getLoan(loanId);
        console.log(
            "collateral after partial repay",
            loanDetailsAfter.collateralAmount
        );

        assertEq(
            loanDetailsAfter.collateralAmount,
            loanDetailsBefore.collateralAmount
        );

        // auction and bidding...
    }

```
###  Output

```solidity
[PASS] test_seizeBorrowerCollateralForLessDebt() (gas: 799390)
Logs:
  Total debt after 10 days of taking loan 1002737850787132101300
  rePaying half  501368925393566050650
  collateral before partial repay 100000000000000000000
  -----------------------------------------
  Total debt after partial repay 501368925393566050650
  collateral after partial repay 100000000000000000000

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 7.09ms
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

## Recommendation

Do not allow `partialRepay` and `addCollateral` after the loan is called.

## [G-1] `LendingTermOffboarding:offBoard` can be called immediately after the quorum is met.
### Code Line

https://github.com/volt-protocol/ethereum-credit-guild/blob/4d33abf95fee69391af0652e3cbe5e0cffa25f9f/src/governance/LendingTermOffboarding.sol#L139

### Details

When offboarding, it is necessary to meet a specific quorum first. Once this requirement is fulfilled, the term can be offboarded and state changes can be made. This prevents any borrowing or other user actions. The `offboard` function is responsible for facilitating this process. It can be invoked by anyone if the `canOffboard` condition returns true and the moment the quorum is reached in `supportOffboard`.

```solidity

if (_weight + userWeight >= quorum) {

canOffboard[term] = true;

}

```

Instead of only setting `canOffboard` to true when a quorum is reached, it can directly call `offBoard` immediately. This optimization helps save a significant amount of gas for the caller of the `offboard` function and also eliminating the need for an external dependency when making the `offboard` function call.

To do this `offBoard` should be made an internal function instead of external.


```diff
function supportOffboard(
        uint256 snapshotBlock,
        address term
    ) external whenNotPaused {
        require(
            block.number <= snapshotBlock + POLL_DURATION_BLOCKS,
            "LendingTermOffboarding: poll expired"
        );
        uint256 _weight = polls[snapshotBlock][term];
        require(_weight != 0, "LendingTermOffboarding: poll not found");
        uint256 userWeight = GuildToken(guildToken).getPastVotes(
            msg.sender,
            snapshotBlock
        );
        require(userWeight != 0, "LendingTermOffboarding: zero weight");
        require(
            userPollVotes[msg.sender][snapshotBlock][term] == 0,
            "LendingTermOffboarding: already voted"
        );

        userPollVotes[msg.sender][snapshotBlock][term] = userWeight;
        polls[snapshotBlock][term] = _weight + userWeight;
        if (_weight + userWeight >= quorum) {
            canOffboard[term] = true;
+            offBoard(term);
        }
        emit OffboardSupport(
            block.timestamp,
            term,
            snapshotBlock,
            msg.sender,
            userWeight
        );
    }
```

```diff
- function offboard(address term) external whenNotPaused {
+ function offboard(address term) internal whenNotPaused {
        require(canOffboard[term], "LendingTermOffboarding: quorum not met");

        // update protocol config
        // this will revert if the term has already been offboarded
        // through another mean.
        GuildToken(guildToken).removeGauge(term);

        // pause psm redemptions

        if (
            nOffboardingsInProgress++ == 0 &&
            !SimplePSM(psm).redemptionsPaused()
        ) {
            SimplePSM(psm).setRedemptionsPaused(true);
        }

        emit Offboard(block.timestamp, term);
    }
```

