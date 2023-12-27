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



