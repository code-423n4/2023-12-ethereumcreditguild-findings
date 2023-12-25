# 1, Checking quorum offboard should minus one

##Lines of code
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L104
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L138

## Details
When propose offboard, voting power start with 1 wei of voting power to initialize the storage slot:

        polls[block.number][term] = 1; // voting power
But when checking quorum, it does not minus that 1 wei of token:

        if (_weight + userWeight >= quorum) {
            canOffboard[term] = true;
        }

## Impact
Wrong number of voting power is calculated. Submited as low since it do not make big impact (1 wei of voting power)

##Mitigration step:
Checking condition should be updated to:

    -    if (_weight + userWeight >= quorum) {
    +    if (_weight + userWeight - 1 >= quorum) {
            canOffboard[term] = true;
        }



# 2, When partial repay, user can not reduce position equal to minBorrow

##Lines of code
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L527-#L531

## Details
When user try to partial repay, in `_partialRepay`, there is a checking condition to check if new borrow amount left is below minBorrow or not:

        require(
            borrowAmount - issuanceDecrease > ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );
The checking condition should use >= instead of >, like in `_borrow()` function:

        require(
            borrowAmount >= ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: borrow amount too low"
        );

## Impact
User can not partial repay borrow that equal to minBorrow

##Mitigration step:
Checking condition should be updated to:

        require(
        -   borrowAmount - issuanceDecrease > ProfitManager(refs.profitManager).minBorrow(),
        +   borrowAmount - issuanceDecrease >= ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );

#3, Offboard proposal can be exprided sooner than expected on L2s

##Lines of code:
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L36

## Vulnerability details
On L2s chain like Optimism, `block.number` can be different from mainnet. For example: optimism is [2 seconds](https://docs.optimism.io/chain/differences). In the `LendingTermOffboarding`, it's constant value:

    uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

In arbitrum, despite it have same block as mainnet, there will be delay in one minute [link](https://docs.arbitrum.io/for-devs/concepts/differences-between-arbitrum-ethereum/block-numbers-and-time)
## Impact
Offboard pool can be expired different than expected on L2s chain, which is not true conpare to documents. Although none of other L2s are mentioned except arbitrum, it will become problem if protocol deployed at other L2s like Optimism

## Tools Used
Manual review

## Recommended Mitigation Steps
It should use time to check instead of block.number
