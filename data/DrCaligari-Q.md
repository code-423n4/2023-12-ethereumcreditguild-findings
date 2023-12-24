File: governance/GuildGovernor.sol

function _setQuorum(uint256 newQuorum) internal {
        emit QuorumUpdated(_quorum, newQuorum);
        _quorum = newQuorum;
}

In the  **_setQuorum** method the event **QuorumUpdated** should be emited after the ** _quorum**  is assigned to **newQuorum** 


https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L68