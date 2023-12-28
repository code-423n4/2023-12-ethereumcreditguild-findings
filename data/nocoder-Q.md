## LendingTermOffboarding.sol: setQuorom() & other setter functions throughout contracts
For the setter functions with privileged access, add common sense limits on the range of inputs that can be given - e.g., should the governor role be allowed to set the required quorum for lending offboarding to 100% because currently they could do that (see below and [code here](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L78C5-L78C5))? This can give more confidence to users that they aren't going to put their funds into your protocol and then face some crazy change. This goes for all setter functions.

```
    function setQuorum(
        uint256 _quorum
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        emit QuorumUpdated(quorum, _quorum);
        quorum = _quorum;
    }
```

## LendingTerm.sol: _borrow()
Why not call the `debtCeiling` function instead of putting this bespoke calculation of it in `_borrow`? The `debtCeiling` function deals with a bunch of edge cases that may be unlikely to happen but you never know, so for consistency's sake, I would recommend just calling `debtCeiling()` instead of doing it this way:  
```
// check the debt ceiling
        uint256 totalBorrowedCredit = ProfitManager(refs.profitManager)
            .totalBorrowedCredit();
        uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
            .gaugeWeightTolerance();
        uint256 _debtCeiling = (GuildToken(refs.guildToken)
            .calculateGaugeAllocation(
                address(this),
                totalBorrowedCredit + borrowAmount
            ) * gaugeWeightTolerance) / 1e18;
        if (totalBorrowedCredit == 0) {
            // if the lending term is deprecated, `calculateGaugeAllocation` will return 0, and the borrow
            // should revert because the debt ceiling is reached (no borrows should be allowed anymore).
            // first borrow in the system does not check proportions of issuance, just that the term is not deprecated.
            require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");
        } else {
            require(
                _postBorrowIssuance <= _debtCeiling,
                "LendingTerm: debt ceiling reached"
            );
        }
```

## SimplePSM.sol: constructor
You should check that ERC20(_pegToken).decimals() isn’t more than 18 or the [last two lines of code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L75) in this constructor would revert…not likely but possible if you don’t whitelist only specific tokens:
```
constructor(
        address _core,
        address _profitManager,
        address _credit,
        address _pegToken
    ) CoreRef(_core) {
        profitManager = _profitManager;
        credit = _credit;
        pegToken = _pegToken;

        uint256 decimals = uint256(ERC20(_pegToken).decimals());
        decimalCorrection = 10 ** (18 - decimals);
    }
```


## LendingTermOnboarding.sol: createTerm()
The note in the following portion of the function says you can set interest to 100% but this require statement does not allow you to set interest to 100% (should be less than or equal to). (Also, consider why interest can't be more than 100% if you're really going for a protocol with a lot of freedom and a large variety of collateral types.) [link to code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L127)
```
require(
            params.interestRate < 1e18, // interest rate [0, 100[% APR
            "LendingTermOnboarding: invalid interestRate"
        );
```


## ERCRebaseDistributor.sol: _burn(), _mint(), _transfer(), _transferFrom
Regarding [your note](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L457) that entering rebase and exiting rebase before _burn, _mint, _transfer, and _transferFrom is cleaner but costs extra gas versus doing the complex calculations in these functions to keep the user rebasing throughout, if they burn or transfer their full amount of CREDIT, they will have to call exitRebase anyway which will cost just as much gas in the end. Granted it saves some gas for someone doing a partial transfer or burn. But I think a full transfer or burn would be common, so I would recommend checking in these functions if the person burned or transferred all their tokens and, if they did, I would change their rebasingState.isRebasing to 0 to save that extra call to exitRebase.

Add this to these functions at the end before you update their rebasingState:
```
if (fromSharesAfter == 0) {
rebasingState[account] = ({
isRebasing: 0,
nshares: 0})
```

## AuctionHouse.sol: startAuction()
The following error shouldn't be “loan previously called” because it could also be loan never called…just use “loan not called this block” or “not called this block” [link to code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L86):
```
     LendingTerm.Loan memory loan = LendingTerm(msg.sender).getLoan(loanId);
        require(
            loan.callTime == block.timestamp,
            "AuctionHouse: loan previously called"
        );
```


## LendingTermOnboarding.sol: createTerm()
Consider input validation on this code for setting partial payments...is there really a reason people should be able to create a term that requires prepayment every second? I could see a scenario where people think it's funny to create something like this and hope some hapless borrower who is out of his depth takes a loan from it...not likely, I know. But if there is a common sense minimum here, I would use it. [link to code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L132)
```
require(
            // 31557601 comes from the constant LendingTerm.YEAR() + 1
            params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]
            "LendingTermOnboarding: invalid maxDelayBetweenPartialRepay"
        );
```

## LendingTermOnboarding.sol: createTerm()
This require statement allows 100% minPartialRepayPercent but that’s just a regular loan. This may be intentional so that people can create loans that actually have a repay date. But if that's not the intention, then place limits on this to prevent people from requiring 100% prepayment (or arguably rename it since it isn't a partial prepayment). [link to code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L138)
```
require(
            params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%
            "LendingTermOnboarding: invalid minPartialRepayPercent"
        );
```

## ERC20Gauges.sol: setMaxGauges()
If it is important that max gauges not be too high to prevent a gas DOS attack, as stated in your notes, you should place a maximum into setMaxGauges to ensure that it can never be too high. [link to code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L444)
```
    function _setMaxGauges(uint256 newMax) internal {
        require(newMax < 1000), “new max too high”);
uint256 oldMax = maxGauges;
        maxGauges = newMax;

        emit MaxGaugesUpdate(oldMax, newMax);
    }
```

## LendingTerm.Sol: onBid()
Move this require statement to the beginning of the else block - it is cleaner and no reason to run more calculations if this require would fail [link to code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L764)
```
 else {
            pnl = int256(creditFromBidder) - int256(principal);
            principal = creditFromBidder;
            require(
                collateralToBorrower == 0,
                "LendingTerm: invalid collateral movement"
            );
        }
```
## ProfitManager.sol: donateToSurplusBuffer()
Just increment the variable surplusBuffer instead of creating a new variable newSurplusBuffer [link to code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol):
```
function donateToSurplusBuffer(uint256 amount) external {
        CreditToken(credit).transferFrom(msg.sender, address(this), amount);
        uint256 newSurplusBuffer = surplusBuffer + amount;
        surplusBuffer = newSurplusBuffer;
        emit SurplusBufferUpdate(block.timestamp, newSurplusBuffer);
    }
```

## ERC20Gauges.sol: incrementGauges()
Move the require statement earlier in this block of [this code](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L285) because it is cleaner and to avoid unnecessary calculations:

```
        // Update gauge specific state
        for (uint256 i = 0; i < size; ) {
            address gauge = gaugeList[i];
            uint256 weight = weights[i];
            weightsSum += weight;

            require(isGauge(gauge), "ERC20Gauges: invalid gauge");
```