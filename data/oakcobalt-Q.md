### Low-01 : Current implementation of `renounceRole()` allows msg.sender to renounce key governor role and admin role, without ensuring roles are granted to other addresses first. In the case of governor keys compromised, key governor functions are at risk of DOS
In src/core/Core.sol,  at the time of deployment, key governor roles are granted to the deployer. The deployer will renounce their roles later on to multi-sig contracts.  But before renouncing their role, deployer will need to grant the governor role to the multi-sig first.

In the case of, a deployer key compromised or oversight during the `renounceRole()`  process, the key governor role account is at risk of being deleted to `address(0)`.

`
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
`
Recommendations:
Consider In `renounceRole(`), also check the member count for a role through `getRoleMemberCount()`. and ensure the count is greater than 1. 

