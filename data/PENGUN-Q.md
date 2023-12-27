## [L-01] `POLL_DURATION_BLOCKS`, `votingPeriod` values are set differently than intended.


## Impact


The `POLL_DURATION_BLOCKS` is set shorter than intended, which could cause offboarding polls to last shorter.


## Details


According to etherscan, the average block time is 12.08 seconds. (https://etherscan.io/chart/blocktime)


However, the current `POLL_DURATION_BLOCKS` is based on a calculation of 7 days, assuming 1 block is 13 seconds.


So, if we actually calculate this value, it comes out to 6.5 days.

It is also recommended to use `BLOCK TIME` as a configurable variable rather than a fixed value, as it can change.

## Recommendation


Set `POLL_DURATION_BLOCKS` to 50066 considering the average block time.


## [L-02] The implementation of `nonRebasingSupply` is incorrect.


## Impact.


The `nonRebasingSupply` does not properly return the non rebasing supply.


## Details


```solidity
/// @notice Total number of the tokens that are not rebasing.
    function nonRebasingSupply() external view virtual returns (uint256) {
        uint256 _totalSupply = totalSupply();
        uint256 _rebasingSupply = rebasingSupply();


        // compare rebasing supply to total supply :
        // rounding errors due to share price & number of shares could otherwise
        // make this function revert due to an underflow
        if (_rebasingSupply > _totalSupply) {
            return 0;
        } else {
            return _totalSupply - _rebasingSupply;
        }
    }
```


The `nonRebasingSupply` returns `totalSupply - rebasingSupply`.


In this case, rebasingSupply() is the value of all `totalRebasingShares` changed to balance based on the current `rebasingSharePrice`.


Note that this figure includes quantities that have not actually been minted, so subtracting them from totalSupply will cause nonRebasingSupply to be smaller than it actually is.


## Recommendation


It is recommended to add the unminted quantity to totalSupply and then subtract rebasingSupply.


## [L-03] SimplePSM does not support pegTokens with decimals greater than 18


## Impact


PegTokens with decimals greater than 18 cannot be used in SImplePSM.


## Details


```solidity
constructor(
        address _core,
        address _profitManager,
        address _credit,
        address _pegToken
    ) CoreRef(_core) { // const
        profitManager = _profitManager; // set pm
        credit = _credit; // set credit
        pegToken = _pegToken; // pegtoken setting


        uint256 decimals = uint256(ERC20(_pegToken).decimals()); // get decimals
        decimalCorrection = 10 ** (18 - decimals); //set correction @audit underflow
    }
```


If decimals is greater than 18, it will be reverted when calculating decimalCorrection.


## Recommendation


Use decimalCorrection as an int type.


## [L-04] Missing slippage check in repay.


## Impact


A larger amount may be repaid than the user intended.


## Details


The quantity the user should repay is calculated by the expression below


`loanDebt = (loanDebt * loan.borrowCreditMultiplier) / creditMultiplier;`


The creditMultiplier will decrease when a loss occurs, so the user will have to repay a larger amount when a loss occurs.


If a large amount of losses occurred in the block where the user sent the repayment request, causing the loanDebt to rise significantly, the user will repay a larger amount than intended.


## Recommendation


It is recommended to check repay by accepting amountRepay as an argument.


## [L-05] MIN_STAKE may not work properly.


## Impact


The value of MIN_STAKE is getting smaller and smaller, so it may not be able to do its job.


## Details


The value of Credit may decrease over time.


Therefore, MIN_STAKE, which is fixed at 1e18, may become less and less valuable and fail to fulfill its original role.


## Recommendation


It is recommended to make the value of MIN_STAKE a modifiable variable rather than a constant.