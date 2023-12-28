# Overview of the Ethereum Credit Guild

Ethereum Credit Guild is a protocol for trust minimized pooled lending. Borrower can deposit collateral token to borrow credit token, and lender can deposit peg token to get credit token. Credit token then can be used to enter rebase or stake to get guild token for voting

# Mechanism of protocol:

- Protocol rely on off-chain alot. Users will tracking healthy of position and voting off a term if positions of borrower underwater if borrowers do not maintain a healthy position by add collateral
- Distribute profit: after user repay/partial repay, profit will be distributed for surplus buffer, guild staker, credit rebasor and other address with the rate decided by governor
- If a bad debt happen, `creditMultiplier` will be reduced, and it will never increase again. It will make a bad effect for all user in the gauge. It is incentive for borrowers and lenders to keep all position healthy
- Credit rebasing: reward that sent to rebasor is distributed to all rebasor in 30 days, to avoid flashloan attack
- GUILD token: user can stake credit token to get GUILD token, which is used to voting and claim reward
- Rate limiting: it is used to limit minting too much token at a small of time

# Approach:
- I started reading document to understand what is happening at the protocol, roles, the idea and mechanism of each part
- After that, i started digging at governance mechanism, and understand about mechanism: timelock, veto, onboard, offboard, voting, ...
- Then, i go to the rest of contract and understand workflow of user when interacting with protocol. This part is the most complexity because there are many new mechanism that need to understand

# Centralization risk:
`GUARDIAN` role, managed by multisig of the team, can pause/unpause some actions to stop protocol.  But most of the important functions is decided by governor, which make this protocol decentralization.

# Other risks:
- System rely too much on off-chain actions like tracking position, call auction loan, apply gauge loss, ..
- Veto mechanism: if a malicious veto have enough voting power, he will be able to veto anyall critical proposals, such as for updating implementations or withdrawing collateral from forgive debt, can be vetoed so the negative impact can be very high.

# Conclusion:
In summary, while Ethereum credit guild promise as a lending protocol, it requires further enhancements in addressing several issues that mentioned above.





### Time spent:
50 hours