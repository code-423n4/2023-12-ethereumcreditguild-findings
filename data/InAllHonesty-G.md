## [G-01] `LendingTerm.debtCeiling()` can be optimized

By placing the cached SLOAD later in the execution flow, we create a possibility of not caching it in case the `if (totalBorrowedCredit == 0 && gaugeWeight != 0)` yields true and returns thus halting execution, thereby enhancing the gas efficiency of the  `debtCeiling()` function.

```diff
--      uint256 _issuance = issuance; // cached SLOAD @audit-info GAS this can be moved to improve efficiency, maybe we hit the below condition and return and we don't need to cache this
        uint256 totalBorrowedCredit = ProfitManager(refs.profitManager)
            .totalBorrowedCredit();
        uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
            .gaugeWeightTolerance();
        if (totalBorrowedCredit == 0 && gaugeWeight != 0) {
            // first-ever CREDIT mint on a non-zero gauge weight term
            // does not check the relative debt ceilings
            // returns min(hardCap, creditMinterBuffer)
            return
                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
        }
++      uint256 _issuance = issuance; // cached SLOAD
        uint256 toleratedGaugeWeight = (gaugeWeight * gaugeWeightTolerance) / 1e18;
        uint256 debtCeilingBefore = (totalBorrowedCredit *
            toleratedGaugeWeight) / totalWeight; 
        if (_issuance >= debtCeilingBefore) {
            return debtCeilingBefore; // no more borrows allowed
        }
--      uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0 @audit-info GAS this can be moved to improve efficiency, maybe we hit the below condition and return and we don't need to initialize this
        if (toleratedGaugeWeight >= totalWeight) {
            // if the gauge weight is above 100% when we include tolerance,
            // the gauge relative debt ceilings are not constraining.
            return
                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
        }
++      uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0 
        uint256 otherGaugesWeight = totalWeight - toleratedGaugeWeight; // always >0
```

## [G-02] `LendingTerm.getLoanDebt()` can be optimized

The `interest` variable can be unpacked into the `loanDebt` because `interest` is used only once in that function.

```diff
        uint256 borrowAmount = loan.borrowAmount;
--      uint256 interest = (borrowAmount *  //@audit-info GAS unpack interest variable into loanDebt, it's used only once
--          params.interestRate *
--          (block.timestamp - borrowTime)) /
--          YEAR /
--            1e18;
--      uint256 loanDebt = borrowAmount + interest;
++      uint256 loanDebt = borrowAmount + (borrowAmount * params.interestRate * (block.timestamp - borrowTime)) / YEAR / 1e18; 
        uint256 _openingFee = params.openingFee;
```

`Overall gas change: (gas: -32 (-0.004%))`

## [G-03] `SurplusGuildMinter.stake()` can be optimized

By moving the `require(amount >= MIN_STAKE, "SurplusGuildMinter: min stake")` condition before we can save gas in case the amount is < MIN_STAKE, because we don't perform the getRewards() function and the other `require(lastGaugeLoss != block.timestamp, "SurplusGuildMinter: loss in block");`

```diff
    /// @notice stake CREDIT tokens to start voting in a gauge.
    function stake(address term, uint256 amount) external whenNotPaused {
++      require(amount >= MIN_STAKE, "SurplusGuildMinter: min stake");
        // apply pending rewards
        (uint256 lastGaugeLoss, UserStake memory userStake, ) = getRewards(
            msg.sender,
            term
        );
        
        require(
            lastGaugeLoss != block.timestamp,
            "SurplusGuildMinter: loss in block"
        );

--      require(amount >= MIN_STAKE, "SurplusGuildMinter: min stake");

        // pull CREDIT from user & transfer it to surplus buffer
        CreditToken(credit).transferFrom(msg.sender, address(this), amount);
```
`Overall gas change: (gas: -6312 (-1.148%))`

## [G-04] `LendingTerm._partialRepay` can be optimized

The `principal` variable can be unpacked into the `principalRepaid` because `principal` is used only once in the `_partialRepay` function.

```diff
        uint256 borrowAmount = loan.borrowAmount;
        uint256 creditMultiplier = ProfitManager(refs.profitManager)
            .creditMultiplier();
--      uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /
            creditMultiplier;
--      uint256 principalRepaid = (principal * percentRepaid) / 1e18;
++      uint256 principalRepaid = ((borrowAmount * loan.borrowCreditMultiplier) / creditMultiplier) * percentRepaid / 1e18;
        uint256 interestRepaid = debtToRepay - principalRepaid;
        uint256 issuanceDecrease = (borrowAmount * percentRepaid) / 1e18;
        require(
            principalRepaid != 0 && interestRepaid != 0,
            "LendingTerm: repay too small"
        );
```

`Overall gas change: (gas: -60 (-0.012%))`

## [G-05] `SurplusGuildMinter.unstake()` can be optimized

The `userMintRatio` variable is used only once. It can be unpacked into the `guildAmount` in order to save gas.

```
--      uint256 userMintRatio = (uint256(userStake.guild) * 1e18) /
            userStake.credit; /// upcast guild to prevent overflow
--      uint256 guildAmount = (userMintRatio * amount) / 1e18;
++      uint256 guildAmount = ((uint256(userStake.guild) * 1e18) /
            userStake.credit * amount) 
```
`Overall gas change: (gas: -15 (-0.002%))`

## [G-06] `GuildVetoGovernor.state()` can be optimized

The two checks `status == ProposalState.Pending` and `status == ProposalState.Executed` should be moved before `queueid` is declared, doing so we save gas in case one of the above mentioned checks yields true and the execution ends in one of the return statements.

```
    function state(
        uint256 proposalId
    ) public view override returns (ProposalState) {
        ProposalState status = super.state(proposalId);

++      // Proposal already executed and stored in state
++      if (status == ProposalState.Executed) {
++          return ProposalState.Executed;
++      }
++      // Proposal cannot be Canceled because there is no public cancel() function.
++      // Vote has just been created, still in waiting period
++      if (status == ProposalState.Pending) {
++          return ProposalState.Pending;

        bytes32 queueid = _timelockIds[proposalId];

        // @dev all proposals that are in this Governor's state should have been created
        // by the createVeto() method, and therefore should have _timelockIds set, so this
        // condition check is an invalid state that should never be reached.
        assert(queueid != bytes32(0));

--      // Proposal already executed and stored in state
--      if (status == ProposalState.Executed) {
--          return ProposalState.Executed;
--      }
--      // Proposal cannot be Canceled because there is no public cancel() function.
--      // Vote has just been created, still in waiting period
--      if (status == ProposalState.Pending) {
--          return ProposalState.Pending;
        }

```