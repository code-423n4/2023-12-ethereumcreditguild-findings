## L-01 The code assumes `block.timestamp` will be an exact value. But this can vary across nodes.

## Impact

Logical failures due to invalid assumptions about deterministic `block.timestamp` values. [SurplusGuildMinter.sol#L122](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L122-L122)

```solidity
            lastGaugeLoss != block.timestamp,
```
Comparing block timestamps precisely enables potential manipulation. 

Miners can adjust block timestamps within an approximately 900 second range. 

If network time jitter or miner manipulation causes a small `block.timestamp` change - even 1 second - this strict equality check would fail unexpectedly.

## Recommendation

Use timestamp ranges, not strict equalities:

```solidity
lastUpdateTimestamp > (block.timestamp - 10 minutes)
```


## L-02 The `initializeReferences` function sets critical token contract references but does not emit any events.

## Impact

Unable to conveniently monitor or react to changes in key variable assignments. Manual storage inspection would be required instead. [ProfitManager.sol#initializeReferences](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L156-L167)

```solidity
function initializeReferences(
    address _credit,
    address _guild,
    address _psm
) external onlyCoreRole(CoreRoles.GOVERNOR) {
    assert(
        credit == address(0) && guild == address(0) && psm == address(0)
    );
    credit = _credit;
    guild = _guild;
    psm = _psm;
}
```
No `emit` statements on storage state mutations that track contract address references. This lacks visibility into storage changes for off-chain consumers.

## Recommendation

Emit events on each reference assignment
```solidity
emit CreditUpdated(_credit);
emit GuildUpdated(_guild); 
emit PSMUpdated(_psm);
```
This best practice increases observability for off-chain applications so they can reliably track contract state changes.


## L-03 The `mint` function issues new tokens but does not emit any event

## Impact

Inability to conveniently track minting transactions occurring in the contract. Would require more expensive manual state inspection. [RateLimitedMinter.sol#mint](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/rate-limits/RateLimitedMinter.sol#L49-L55)

No `emit` statement on the critical token minting operation.

```solidity
function mint(
    address to,
    uint256 amount
) external onlyCoreRole(role) whenNotPaused {
    _depleteBuffer(amount); /// check and effects
    IERC20Mintable(token).mint(to, amount); /// interactions
}
```
This reduces visibility into token supply changes.

## Recommendation

Emit a Mint event on supply increase


## L-04 The `replenishBuffer` function also modifies important protocol state without emitting any event.

## Impact

Inability to conveniently track buffer replenishments occurring in the contract. Manual state inspection would be required instead. [RateLimitedMinter.sol#replenishBuffer](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/rate-limits/RateLimitedMinter.sol#L60-L62)

```solidity
function replenishBuffer(uint256 amount) external onlyCoreRole(role) {
    _replenishBuffer(amount); /// effects
}
```
This buffer tracks available supply for future minting. Changes can reduce transparency.

## Recommendation

Emit a BufferReplenish event on change

```solidity
function replenishBuffer(uint256 amount) external {

  _replenishBuffer(amount);
  
  emit BufferReplenish(amount, block.timestamp); 

}
```
This best practice increases visibility into key operating metrics for off-chain consumers.

## L-04 Several functions in the `CreditToken` contract change critical governance parameters or token supply but do not emit any events

## Impact

Unable to conveniently monitor token supply inflation, permission changes, rebasing updates etc occurring within the contract. [CreditToken.sol#mint](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L35-L40) [reditToken.sol#setMaxDelegates](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L43-L47) [CreditToken.sol#forceEnterRebase](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L58-L66) [CreditToken.sol#_burn](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L90-L96)

```solidity
function mint(
    address to,
    uint256 amount
) external onlyCoreRole(CoreRoles.CREDIT_MINTER) {
    _mint(to, amount);
}

function setMaxDelegates(
    uint256 newMax
) external onlyCoreRole(CoreRoles.CREDIT_GOVERNANCE_PARAMETERS) {
    _setMaxDelegates(newMax);
}

function forceEnterRebase(
    address account
) external onlyCoreRole(CoreRoles.CREDIT_REBASE_PARAMETERS) {
    require(
        rebasingState[account].isRebasing == 0,
        "CreditToken: already rebasing"
    );
    _enterRebase(account);
}

function _burn(
    address account,
    uint256 amount
) internal override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor) {
    _decrementVotesUntilFree(account, amount); // from ERC20MultiVotes
    ERC20RebaseDistributor._burn(account, amount);
}
```
Lack of `emit` statements for storage mutations that impact user balances and governance rights.

## Recommendation

Emit events on each state change

```solidity
function mint(address to, uint256 amount) external {
  _mint(to, amount);
  emit Mint(to, amount);  
}

function _burn(address who, uint256 amount) internal {
  super._burn(who, amount);
  emit Burn(who, amount);
} 
```
This best practice increases transparency for off-chain services tracking the contract.


## L-06 There are a few functions that specify a return type but do not explicitly return a value.

```solidity
function propose() public pure override returns (uint256) {
    revert("LendingTermOnboarding: cannot propose arbitrary actions"); 
}

function nonRebasingSupply() external view virtual returns (uint256) {
   // Logic to calculate value
   // But missing return statement  
}
```
The function declarations promise to return a value per the `returns` statement. But no `return x;` explicitly sends back a value.

## Impact

This forces the functions to always return the default data type value despite their internal logic:

* `propose`: always returns 0 despite reverting
* `nonRebasingSupply`: returns 0 instead of calculated amount

[GuildVetoGovernor.sol#propose](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L304-L311) | [ERC20RebaseDistributor.sol#nonRebasingSupply](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L399-L411)
```solidity
function propose(
    address[] memory /* targets*/,
    uint256[] memory /* values*/,
    bytes[] memory /* calldatas*/,
    string memory /* description*/
) public pure override returns (uint256) {
    revert("GuildVetoGovernor: cannot propose arbitrary actions");
}

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

## Recommendation

Explicitly return a value matching the expected type.

```solidity
function propose() public pure override returns (uint256) {
  revert("LendingTermOnboarding: cannot propose arbitrary actions");
  
  // Explicitly return 0
  return 0;
}
```

## L-07 The `state` and `balanceOf` functions declare return types but do not explicitly return

```solidity
function state(uint256 proposalId) public view override returns (ProposalState) {

  // Logic determining status

  // Missing return statement
  
}

function balanceOf(address account) public view override returns (uint256) {

  // Logic calculating balance
  
  // Does not return balance
  
}
```
## Impact

While these functions promise a return value per their signature, no `return x;` statement sends back the expected result.

Forces incorrect default data despite logic:

* `state`: returns initial ProposalState value, not actual
* `balanceOf`: returns 0 instead of calculated balance

[GuildVetoGovernor.sol#state](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L246-L301) | [ERC20RebaseDistributor.sol#balanceOf](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L426-L441)

```solidity
function state(
    uint256 proposalId
) public view override returns (ProposalState) {
    ProposalState status = super.state(proposalId);
    bytes32 queueid = _timelockIds[proposalId];


    // @dev all proposals that are in this Governor's state should have been created
    // by the createVeto() method, and therefore should have _timelockIds set, so this
    // condition check is an invalid state that should never be reached.
    assert(queueid != bytes32(0));


    // Proposal already executed and stored in state
    if (status == ProposalState.Executed) {
        return ProposalState.Executed;
    }
    // Proposal cannot be Canceled because there is no public cancel() function.
    // Vote has just been created, still in waiting period
    if (status == ProposalState.Pending) {
        return ProposalState.Pending;
    }


    // at this stage, status from super can be one of: Active, Succeeded, Defeated
    // Read timestamp in the timelock to determine the state of the proposal
    uint256 timelockOperationTimestamp = TimelockController(
        payable(timelock)
    ).getTimestamp(queueid);


    // proposal already cleared from the timelock by something else
    if (timelockOperationTimestamp == 0) {
        return ProposalState.Canceled;
    }
    // proposal already executed in the timelock
    if (timelockOperationTimestamp == 1) {
        return ProposalState.Defeated;
    }


    // proposal still in waiting period in the timelock
    if (timelockOperationTimestamp > block.timestamp) {
        // ready to veto
        // no need for "&& _voteSucceeded(proposalId)" in condition because
        // veto votes are always succeeded (there is no tallying for 'for'
        // votes against 'against' votes), the only condition is the quorum.
        if (_quorumReached(proposalId)) {
            return ProposalState.Succeeded;
        }
        // need more votes to veto
        else {
            return ProposalState.Active;
        }
    }
    // proposal is ready to execute in the timelock, the veto
    // vote did not reach quorum in time.
    else {
        return ProposalState.Defeated;
    }
}


function balanceOf(
    address account
) public view virtual override returns (uint256) {
    RebasingState memory _rebasingState = rebasingState[account];
    if (_rebasingState.isRebasing == 0) {
        return ERC20.balanceOf(account);
    } else {
        return
            _shares2balance(
                _rebasingState.nShares,
                rebasingSharePrice(),
                0,
                ERC20.balanceOf(account)
            );
    }
}
```

## Recommendation

Explicitly return the expected type

```solidity
function state(uint256 proposalId) public view override returns (ProposalState) {

  ProposalState status = // determined status
  
  return status;

}
```
This aligns internal logic with external return value expectations.

## L-08 `ProfitManager` contract does make external token calls before updating state

## Impact

Calling external contracts before updating state variables. This introduces reentrancy risk if the token were to call back into `ProfitManager` before `donateToSurplusBuffer` finishes.

[ProfitManager.sol#donateToSurplusBuffer](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L251-L256)
```solidity
function donateToSurplusBuffer(uint256 amount) external {
    CreditToken(credit).transferFrom(msg.sender, address(this), amount);
    uint256 newSurplusBuffer = surplusBuffer + amount;
    surplusBuffer = newSurplusBuffer;
    emit SurplusBufferUpdate(block.timestamp, newSurplusBuffer);
}
```
Reentrancy could allow manipulating `surplusBuffer` balance, creating accounting inaccuracies, and violating expected control flow.

For example:

* Bob donates tokens | * `ProfitManager` receives tokens | * Token contract calls `ProfitManager` again | * Previous `donateToSurplusBuffer` invocation still in progress | * `surplusBuffer` erroneously increments again

## Recommendation

Follow checks-effects-interactions pattern

```solidity
function donateToSurplusBuffer(uint256 amount) external {

  surplusBuffer += amount;

  CreditToken(credit).transferFrom(msg.sender, address(this), amount);
  
}
```