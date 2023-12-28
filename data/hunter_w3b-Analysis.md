# Analysis - Ethereum-Credit-Guild Contest

![ECG-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2Fp7vGZ7j3SAY.0&w=256&q=75)

## Description overview of The Ethereum-Credit-Guild Contest

The Ethereum Credit Guild protocol aims to provide decentralized access to borrowing and lending while minimizing reliance on centralized entities like oracles.It allows users to deposit stablecoins like USDC in exchange for credit tokens pegged to those stablecoins like gUSDC, which represent their savings and earn interest from loans taken out by borrowers. Borrowers can take loans by depositing asset collateral and receiving the corresponding credit tokens, paying interest on the loans. There are two main token types -GUILD(governance token) and credit tokens that are pegged to stablecoins like gUSDC. GUILD is the governance token that allows holders to vote on protocol parameters like lending terms and rates through a liquid model. Voting is secured by quorum requirements and veto periods to prevent abrupt changes. Interest earned from loans is distributed to GUILD voters who stake their tokens to vote on lending terms, credit token holders who provide the lending capital, and a surplus buffer that protects against losses. If borrowers default, the collateral is auctioned and any losses first reduce the surplus buffer before being socialized among credit token holders in a manner that maintains the stablecoin peg. The protocol aims to create a censorship-resistant and trust-minimized system for lending and borrowing digital assets by reducing reliance on centralized third parties through its on-chain governance model and token incentives.

**The key contracts of ECG protocol for this Audit are**:

Auditing the key contracts of the ECG Protocol is essential to ensure the security of the protocol and they form the backbone of the ECG protocol, outlining its governance structure, lending terms, token management. Focusing on them first will provide a solid foundation for understanding the protocol's operation and how it manages user assets and stability. Here's why it's important to audit these specific contracts:

It's important to audit contracts like `Core.sol`, `GuildGovernor.sol`, `GuildTimelockController.sol` and `LendingTermOnboarding.sol` to validate that the governance system and voting process works as intended.

Contracts such as `LendingTerm.sol`, `AuctionHouse.sol` and `ProfitManager.sol` that are central to the lending functionality need to be reviewed carefully. This ensures borrowing, collateral liquidations and profit distributions are performed securely.

The token implementation contracts like `CreditToken.sol`, `GuildToken.sol` and `ERC20Gauges.sol` warrant a thorough analysis to guarantee tokens are properly minted and managed with no exploitation risks.

Rate limiting utilities like `RateLimitedMinter.sol` and the peg stability contract `SimplePSM.sol` also require close evaluation to confirm they uphold the stablecoin peg as designed.

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

- **Core.sol**: This contract serves as the access control system for the ECG protocol, managing roles and permissions with a specific initial setup, including the granting of governance-related roles, and provides functionality to create and update roles, ensuring secure and controlled protocol operations.

- **GuildGovernor.sol**: GuildGovernor contract is an on-chain governance implementation for the Ethereum-Credit-Guild, based on the OpenZeppelin framework, incorporating features such as quorum management, voting settings, timelock control, and guardian-specific actions, with roles and permissions controlled by the Core contract of the protocol.

- **GuildTimelockController.sol**: This contract is an overridden version of OpenZeppelin's governance TimelockController, designed for the Ethereum Credit Guild system, with uniform access control based on roles defined in the Core contract, bypassing the original access control system to minimize code changes and leverage the roles management from the Core contract.

- **LendingTermOffboarding.sol**: Provides utilities for offboarding a LendingTerm within the Ethereum Credit Guild system, allowing GUILD holders to initiate and support removal polls, with a specified quorum, to offboard a term, preventing new loans and restricting GUILD holder voting, followed by the cleanup of roles and permissions associated with the offboarded term.

- **LendingTermOnboarding.sol**: Serves as a Governor with restricted proposal capabilities, acting as a factory to create and initialize LendingTerm contracts, allowing users to propose and vote on the onboarding of LendingTerms, and leveraging a Timelock to introduce a veto mechanism for CREDIT holders.

- **ProfitManager.sol**: This contract manages profits generated in a system, distributing them among stakeholders, handling surplus buffers, and adjusting a credit multiplier to account for losses in the system, while also facilitating the distribution of profits to GUILD holders based on their participation in productive gauges also defines configurations for profit sharing, tracks surplus buffers, and updates a credit multiplier to reflect the system's health.

- **GuildVetoGovernor.sol**: Implements an on-chain governance system for the Ethereum Credit Guild based on the OpenZeppelin implementation. It introduces a specific mechanism for adding veto capabilities, where token holders can create proposals to cancel actions in a linked TimelockController, and the cancellation requires a successful veto vote, meaning that a minimum quorum of "against" votes must be reached also is designed to handle veto proposals and execute the cancellation of actions in the linked TimelockController based on the outcome of the veto vote. The governance system is restricted to holders with a Core GOVERNOR role, and the contract includes functionalities for managing quorum, timelock, and vote counting.

- **ERC20Gauges.sol**: The contract is an abstract implementation of an ERC20 token with an embedded "Gauge" style voting mechanism, allowing token holders to allocate weight to supported gauges, which represent addresses receiving resources periodically or continuously also includes functionality to manage gauge weights, deprecate gauges, and implement security measures.

- **ERC20MultiVotes.sol**: This contract is an abstract implementation of an ERC20 token with multi-delegation voting capabilities, allowing token holders to delegate their voting power to multiple addresses, up to a certain limit defined by maxDelegates per user. The contract includes functions for vote calculations, admin operations such as updating the maximum number of delegates and setting flags for contracts to exceed the maximum delegate limit, as well as delegation logic with associated events.

- **CreditToken.sol**: Representing the debt token of the ECG, implementing various features including burning, voting delegation, rebasing, minting, and governance parameters, with inheritance from OpenZeppelin ERC20 contracts and custom functionalities for handling rebasing and voting delegation.

- **GuildToken.sol**: GuildToken contract representing the governance token of the ECG, with features such as non-transferrable deployment, voting delegation, gauge system for defining debt ceilings on lending terms, loss management, transferability control, minting, and burning.

- **AuctionHouse.sol**: This contract is the auction mechanism for collateral of borrowers in the ECG, facilitating the exchange of collateral for CREDIT to cover outstanding debts, featuring auction phases, bid details, and forgiveness functionality.

- **LendingTerm.sol**: It represents a lending term contract that issues CREDIT debt and escrows collateral assets. The interest rate is non-compounding, and the percentage is expressed per period of one Ethereum year. The contract includes functionalities such as initiating loans, adding collateral, and partially repaying open loans, with events emitted for various lifecycle actions of loans and also manages references to other protocol contracts, such as ProfitManager, GuildToken, AuctionHouse, CreditMinter, and CreditToken, and specifies parameters for the lending term, including collateral type, interest rate, maximum delay between partial repayments, and other relevant parameters.

- **SimplePSM.sol**: Functions as a Peg Stability Module (PSM) designed to mint and redeem the CREDIT token while maintaining a stable peg to an underlying token (pegToken), with the peg's value determined by the ProfitManager's creditMultiplier. It allows users to mint CREDIT by depositing peg tokens and redeem peg tokens by burning CREDIT.

- **SurplusGuildMinter.sol**: The contract is enables the minting of GUILD tokens from CREDIT collateral. Users can stake CREDIT tokens to the surplus buffer of selected lending terms, obtaining reduced capital costs and participating in the gauge voting system without exposure to GUILD token price fluctuations. Minted GUILD tokens from this contract can be used solely to participate in the gauge system, increasing debt ceilings, and earning fees from chosen lending terms. The contract includes functions for staking, unstaking, claiming GUILD rewards, and updating mint and reward ratios, along with events for stake, unstake, GUILD rewards, mint ratio updates, and reward ratio updates.

- **RateLimitedMinter.sol**: This contract facilitates token minting with rate-limiting capabilities, ensuring that all minting operations are controlled through this contract and subject to rate restrictions, featuring pausable minting, buffer management, and access control based on specified roles.

- **RateLimitedV2.sol**: Provides a framework for implementing rate limits on actions such as minting, featuring adjustable rate limits, buffer management, and access control through specific roles.

### The ECG protocol defines several roles used for access control within the ECG.

Here are the roles defined in the `CoreRoles` library:

#### Core roles for access control:

1. **GOVERNOR_ROLE:**

   - Description: The all-powerful role that controls all other roles and protocol functionality.

2. **GUARDIAN_ROLE:**
   - Description: The protector role that can pause contracts and revoke roles in an emergency.

#### Token supply roles:

3. **CREDIT_MINTER_ROLE:**

   - Description: Can mint CREDIT arbitrarily.

4. **RATE_LIMITED_CREDIT_MINTER_ROLE:**

   - Description: Can mint CREDIT within rate limits and cap.

5. **GUILD_MINTER_ROLE:**

   - Description: Can mint GUILD arbitrarily.

6. **RATE_LIMITED_GUILD_MINTER_ROLE:**
   - Description: Can mint GUILD within rate limits and cap.

#### GUILD Token Management:

7. **GAUGE_ADD_ROLE:**

   - Description: Can manage and add new gauges to the system.

8. **GAUGE_REMOVE_ROLE:**

   - Description: Can remove gauges from the system.

9. **GAUGE_PARAMETERS_ROLE:**

   - Description: Can manage gauge parameters (max gauges, individual cap).

10. **GAUGE_PNL_NOTIFIER_ROLE:**

- Description: Can notify of profits and losses in a given gauge.

11. **GUILD_GOVERNANCE_PARAMETERS_ROLE:**

- Description: Can update governance parameters for GUILD delegations.

12. **GUILD_SURPLUS_BUFFER_WITHDRAW_ROLE:**

- Description: Can withdraw from GUILD surplus buffer.

#### CREDIT Token Management:

13. **CREDIT_GOVERNANCE_PARAMETERS_ROLE:**

- Description: Can update governance parameters for CREDIT delegations.

14. **CREDIT_REBASE_PARAMETERS_ROLE:**

- Description: Can update rebase parameters for CREDIT holders.

#### Timelock management:

15. **TIMELOCK_PROPOSER_ROLE:**

- Description: Can propose new actions in timelocks.

16. **TIMELOCK_EXECUTOR_ROLE:**

- Description: Can execute actions in timelocks after their delay.

17. **TIMELOCK_CANCELLER_ROLE:**

- Description: Can cancel actions in timelocks.

These roles provide a structured and modular approach to access control, allowing different functionalities to be assigned to specific roles within the Ethereum Credit Guild system.

## Approach Taken-in Evaluating The ECG Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the ECG Protocol.

    I start with the following contracts, which play crucial roles in the ECG protocol:

    **Main Contracts I Looked At**

                GuildGovernor.sol
                LendingTerm.sol
                CreditToken.sol
                ProfitManager.sol
                SimplePSM.sol
                GuildTimelockController.sol
                ERC20Gauges.sol
                GuildToken.sol
                CreditToken.sol
                AuctionHouse.sol

    Each of these contracts fulfills specific functions, such as access control, on-chain governance, lending terms, collateral management, profit distribution, stability mechanisms, token functionalities, and rate-limited minting, contributing to the overall functionality and security of the ECG Protocol.

    I started my analysis by examining the Governance of the Ethereum-Credit-Guild contracts, which serves as the foundational framework for decision-making and control within the protocol, encompassing key components such as role management, voting mechanisms, and timelock controls across a diverse set of contracts, including `Core.sol`, `GuildGovernor.sol`, `GuildVetoGovernor.sol`, `ERC20Gauges.sol`, `ERC20MultiVotes.sol`, `CreditToken.sol`, `GuildToken.sol`, and `GuildTimelockController.sol`.

    Then, I turned my attention to the lending and borrowing processes within the Ethereum-Credit-Guild protocol. In particular, I closely examined the contracts involved in these operations, such as `LendingTermOffboarding.sol` for managing the offboarding of lending terms, `LendingTermOnboarding.sol` for proposing and voting on the onboarding of new lending terms, `AuctionHouse.sol` for the collateral auction mechanism, and `LendingTerm.sol` for the core representation of lending terms, encompassing functionalities like loan initiation, collateral addition, and partial loan repayments.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://credit-guild.gitbook.io/) for a more detailed and technical explanation of ECG protocol.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the contracts that are part of the ECG protocol:

![Diagram](<https://3428599126-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FfzMdp0yM8XIbpZXK4aNC%2Fuploads%2F2DygJ7VG4fwtXzrE7IG8%2Fecg_smartcontracts.drawio%20(5).png?alt=media&token=1069c6fc-1ed0-43a1-aab9-bde2cf3aff16>)

### Governance Contracts:

- **GuildGovernor.sol**: On-chain governance implementation with features like quorum management, voting settings, and timelock control.
- **GuildTimelockController.sol**: An overridden version of OpenZeppelin's TimelockController, designed for uniform access control in the governance system.

### Lending Contracts:

- **LendingTermOffboarding.sol**: Provides utilities for offboarding lending terms, allowing GUILD holders to initiate removal polls and restrict voting, followed by cleanup.
- **LendingTermOnboarding.sol**: Serves as a Governor for proposing and voting onboarding of new lending terms, incorporating a Timelock and veto mechanism for CREDIT holders.
- **AuctionHouse.sol**: Manages the auction mechanism for collateral, facilitating the exchange of collateral for CREDIT to cover outstanding debts.
- **LendingTerm.sol**: Represents a lending term contract issuing CREDIT debt, handling collateral, and managing various lifecycle actions of loans.

### Profit and Governance Token Management:

- **ProfitManager.sol**: Manages profits generated in the system, distributes them among stakeholders, and adjusts a credit multiplier to reflect the system's health.
- **GuildVetoGovernor.sol**: Implements on-chain governance with a specific mechanism for veto capabilities, restricting actions based on a minimum quorum of "against" votes.
- **ERC20Gauges.sol**: Abstract implementation of an ERC20 token with a "Gauge" style voting mechanism, allowing token holders to allocate weight to supported gauges.

### Voting Mechanisms:

- **ERC20MultiVotes.sol**: Abstract implementation of an ERC20 token with multi-delegation voting capabilities, allowing token holders to delegate voting power to multiple addresses.

### Debt and Governance Tokens:

- **CreditToken.sol**: Represents the debt token of the ECG, incorporating features like burning, voting delegation, rebasing, minting, and governance parameters.
- **GuildToken.sol**: Represents the governance token of the ECG, featuring non-transferrable deployment, voting delegation, and various governance-related functionalities.

### Stability and Minting Contracts:

- **SimplePSM.sol**: Functions as a Peg Stability Module, allowing users to mint and redeem the CREDIT token while maintaining a stable peg to an underlying token.

### GUILD Token Minting and Rewards:

- **SurplusGuildMinter.sol**: Enables the minting of GUILD tokens from CREDIT collateral, allowing users to stake CREDIT tokens to the surplus buffer and participate in the gauge voting system.

### Rate Limiting Contracts:

- **RateLimitedMinter.sol**: Facilitates token minting with rate-limiting capabilities, ensuring controlled operations subject to rate restrictions.
- **RateLimitedV2.sol**: Provides a framework for implementing rate limits on actions such as minting, featuring adjustable rate limits and buffer management.

ECG protocol integrates a sophisticated set of contracts to manage governance, lending and borrowing, profit distribution, voting mechanisms, token management, stability, minting, and rate limiting, creating a robust and comprehensive decentralized finance (DeFi) ecosystem.

### Some potential areas for improvement and Architecture feedback

1. **Governance Enhancements**:

The governance system, implemented through GuildGovernor.sol and associated contracts, could benefit from additional features such as detailed voting analytics and participation incentives to address potential voter apathy.

2. **Quorum Management**:

Consider refining the quorum management in GuildGovernor.sol to strike a balance between achieving a sufficient voter turnout and avoiding undue reliance on paid delegates. Exploring mechanisms to dynamically adjust quorum based on protocol health and activity levels may enhance decision-making efficiency.

3. **Profit Distribution Mechanism**:

ProfitManager.sol, responsible for managing profits and distributing them among stakeholders, could be further optimized for scalability and gas efficiency. Explore strategies to minimize transaction costs and enhance the accuracy of profit distribution.

4. **Auction Mechanism Improvements**:

Analyze the efficiency of AuctionHouse.sol, exploring potential improvements to the collateral exchange mechanism. Consider optimizations to minimize bid details and enhance forgiveness functionality for a more user-friendly experience.

5. The Protocol doesn't include explicit mechanisms for upgradability. If upgrades are necessary, it might require a new deployment, which could be seen as a lack of flexibility.

## Codebase Quality

Overall, I consider the quality of the ECG protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The ECG Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                       |
| **Code Comments**                        | During the audit of the ECG contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                 |
| **Documentation**                        | The documentation of the ECG project is quite comprehensive and detailed, providing a solid overview of how ECG protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 99% this is Good.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last.                                                                                                                                                                                                                                                                                                 |
| **Strengths**                            | 1. Formal Verfication in certora, 2. Comprehensive unit tests and integration tests, 3. Utilization of Natspec                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| **Error**                                | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Imports**                              | In ECG protocol the contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files. 2. openzeppelin contracts updated to a newer version.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the ECG protocol. These risks encompass concentration risk in GuildGovernor, LendingTermOnboarding risk and more, third-party dependency risk, and centralization risks arising.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. The `Core` contract appears to grant the initial governance role (`CoreRoles.GOVERNOR`) to a single address (`msg.sender`). If this address becomes compromised or unavailable, there could be a significant impact on the entire protocol's governance, potentially leading to a halt in decision-making processes.

2. The `CoreRef::emergencyAction` function introduces a systemic risk by allowing the execution of arbitrary calldata against arbitrary addresses. While this flexibility can be useful in emergency scenarios, it also poses a significant risk if misused or if unintended actions are performed. Any vulnerabilities in the executed contracts or unintended consequences could impact the overall system.

3. The `LendingTermOnboarding::proposeOnboard` function dynamically generates proposal data for onboarding a term. If not carefully validated, dynamic proposal generation may lead to unexpected behavior, such as incorrect data being proposed or vulnerabilities related to dynamic data.

4. The `ProfitManager::gaugeWeightTolerance` parameter introduces a tolerance level for new borrows concerning gauge weights.In situations where the tolerance is set too high, it might lead to imbalances and potential issues in the system's borrowing mechanism.

5. The comment mentions in ERC20MultiVotes contract that maxDelegates is critical to protect against gas DOS attacks upon token transfer. The gas limit for a block constrains the number of operations that can be executed. If `maxDelegates` is set too high, it might lead to gas-related issues.

6. The functions `CreditToken::forceEnterRebase` and `CreditToken::forceExitRebase` allow external entities with specific roles (CoreRoles.CREDIT_REBASE_PARAMETERS) to force an address into or out of the rebase mechanism. This could introduce potential risks if not properly controlled and monitored.

7. The GuildToken.sol contract implements a mechanism for notifying and applying losses in gauges, allowing users to slash tokens. The logic around loss management and slashing introduces potential risks if not implemented and managed carefully.

8. The `RateLimitedMinter::mint` function depletes the buffer during token minting and checks for pausing (whenNotPaused). If there are issues with the pause mechanism or if the contract is paused unexpectedly, it may result in the inability to mint tokens even if the buffer is available.

### Centralization Risks:

1. The `Core.sol` contract in the ECG protocol presents a centralization risk since it confers privileged functions like role administration and emergency actions in `CoreRef.sol` to an admin/owner, requiring careful trust that such powers will not be abused or lead to a single point of failure for the system through malicious updates or fund drainage.

2. While the LendingTermOffboarding contract allows any GUILD holder to vote, the influence of an individual voter is proportional to their GUILD holdings. This might lead to scenarios where a small number of large GUILD holders dominate the decision-making process, reducing the influence of smaller holders.

3. The function `CreditToken::_decrementVotesUntilFree` is called during the transfer and transferFrom functions. This could impact the decentralization of voting power, especially if a centralized authority has control over the logic for decrementing votes.

4. Auction information is stored in a mapping (auctions) using loan IDs as keys in AuctionHouse.sol contract. Centralized control over modifying or deleting entries in this mapping could introduce risks if not properly managed.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the ECG protocol.**

## Conclusion

In general, The Ethereum Credit Guild protocol presents a well-designed architecture. The ECG protocol exhibits strengths in formal verification, comprehensive testing, and detailed documentation, we believe the team has done a good job regarding the code. However, there are areas for improvement, including governance features, profit distribution optimization, and code documentation. Systemic risks include initial governance role assignment and dynamic proposal generation, while centralization risks involve certain contracts and functions. Recommendations include enhancing governance features, optimizing profit distribution, improving documentation, addressing systemic risks, and mitigating centralization risks. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
19 hours