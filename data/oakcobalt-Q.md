### Low-01 : Current implementation of `renounceRole()` allows msg.sender to renounce key governor role and admin role, without ensuring roles are granted to other addresses first. In the case of governor keys compromised, key governor functions are at risk of DOS
In src/core/Core.sol,  at the time of deployment, key governor roles are granted to the deployer. The deployer will renounce their roles later on to multi-sig contracts.  But before renouncing their role, deployer will need to grant the governor role to the multi-sig first.

In the case of, a deployer key compromised or oversight during the `renounceRole()`  process, the key governor role account is at risk of being deleted to `address(0)`.

```
    function renounceRole(bytes32 role, address account) public virtual override {
        require(account == _msgSender(), "AccessControl: can only renounce roles for self");
        _revokeRole(role, account);
    }
`
`
    function _revokeRole(bytes32 role, address account) internal virtual {
        if (hasRole(role, account)) {
|>         _roles[role].members[account] = false;
            emit RoleRevoked(role, account, _msgSender());
        }
```

Recommendations:

Consider In `renounceRole(`), also check the member count for a role through `getRoleMemberCount()`. and ensure the count is greater than 1. 

### Low-02: Users can still vote even if a poll already reached the quorum, user will waste their gas in their case, since their transaction doesn't change the poll result.

In src/governance/LendingTermOffboarding.sol- `supportOffboard()`, users can vote to offboard a lending term. However, when a quorum is already reached before the poll expires, a user can still vote through `supportOffboard()`.  The transaction will go through and the state variable will be re-written to the same state as before, leading to user waste gas.

```
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




### NC-01: Incomplete comment.  AccessControlEnumerable also has a public `supportsInterface()`  function

In src/core/Core.sol,  there is a comment listing all the functions available in AccessControl.sol. But the comment is missing `supportsInterface()` function.

```
   // AccessControlEnumerable is AccessControl, and also has the following functions :
    // hasRole(bytes32 role, address account) -> bool
    // getRoleAdmin(bytes32 role) -> bytes32
    // grantRole(bytes32 role, address account)
    // revokeRole(bytes32 role, address account)
    // renounceRole(bytes32 role, address account)
``
Recommendations:
Add `supportsInterface(bytes4 interfaceId)->bool`  in the comment.

