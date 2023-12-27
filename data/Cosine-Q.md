# The `createVeto` function allows the creation of veto proposals against non-existing or canceled actions

GitHub:\
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L319-L322

The `createVeto` function inside `GuildVetoGovernor` does not check if the `timelockExecutionTime` from the `TimelockController` equals 0. Therefore, it is possible to create veto proposals for non-existing or already canceled actions.

```solidity
function createVeto(bytes32 timelockId) external returns (uint256) {
  // Check that the operation is pending in the timelock
  uint256 timelockExecutionTime = TimelockController(payable(timelock)).getTimestamp(timelockId);
  require(timelockExecutionTime > 1, 'GuildVetoGovernor: action must be pending');

  // Build proposal data
  (
    address[] memory targets,
    uint256[] memory values,
    bytes[] memory calldatas,
    string memory description
  ) = _getVetoCalls(timelockId);

  uint256 proposalId = super.propose(targets, values, calldatas, description);

  // Save mapping between proposalId and timelockId
  _timelockIds[proposalId] = timelockId;

  return proposalId;
}

```

Here we can see the `getTimestamp` and `cancel` functions inside the `TimelockController` contract and how calling `cancel` will delete the timestamp and therefore `getTimestamp` will return the default value 0 which is not checked:

```solidity
function getTimestamp(bytes32 id) public view virtual returns (uint256) {
  return _timestamps[id];
}

function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {
  require(isOperationPending(id), 'TimelockController: operation cannot be cancelled');
  delete _timestamps[id];

  emit Cancelled(id);
}

```

# Missing zero value check in `mint` and `redeem` function of the `SimplePSM` contracts can lead to misleading events being emitted

GitHub:\
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L101-L112 \
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L132-L144

The `mint` and `redeem` functions inside `SimplePSM` do not check if the given amount is zero. This can lead to misleading events being emitted as the events will be emitted indicating that a mint or redeem action was executed but in reality nothing happened. This can lead to frontend bugs depending on the implementation.

Here we can see the code of the `mint` and `redeem` functions:

```solidity
function mint(address to, uint256 amountIn) external whenNotPaused returns (uint256 amountOut) {
  amountOut = getMintAmountOut(amountIn);
  pegTokenBalance += amountIn;
  ERC20(pegToken).safeTransferFrom(msg.sender, address(this), amountIn);
  CreditToken(credit).mint(to, amountOut);
  emit Mint(block.timestamp, to, amountIn, amountOut);
}

```

```solidity
function redeem(address to, uint256 amountIn) external returns (uint256 amountOut) {
  require(!redemptionsPaused, 'SimplePSM: redemptions paused');
  amountOut = getRedeemAmountOut(amountIn);
  CreditToken(credit).burnFrom(msg.sender, amountIn);
  pegTokenBalance -= amountOut;
  ERC20(pegToken).safeTransfer(to, amountOut);
  emit Redeem(block.timestamp, to, amountIn, amountOut);
}

```

# Voting delay of zero can lead to bad consequences

GitHub:

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/test/proposals/gips/GIP_0.sol#L88-L89 \
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/test/proposals/gips/GIP_0.sol#L97-L98

The protocol stated out in the C4 description that the deployment script of the protocol, located in test/proposals/gips/GIP_0.sol is also in scope, as protocol deployment/configuration mistakes could be made. The voting delays are set to zero inside this script.

These delay parameters set the time duration between a proposal is submitted and voting starts. This time can be used to react to a proposal and buy more voting power, or delegate / undelegate voting power. Especially undelgating voting power is important if a proposal could potentially be dangerous to the system and the user does not trust anyone else to vote with the user's voting power. Also, if there is a user who is trusted a lot and a lot of users delegates voting power to that user a big centralization risk occurs if the users who delegate can not stop that user from voting with their power.

Therefore, a non-zero voting delay should be used, and minimum checks should be implemented to prevent the delay from being set to zero.
