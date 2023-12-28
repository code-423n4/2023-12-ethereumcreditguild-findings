


## Introduction

The Ethereum Credit Guild (ECG) is an evolution of Volt Protocol, aiming to establish a neutral and censorship-resistant savings and credit system on the Ethereum blockchain. The key innovation lies in unifying minting within a Collateralized Debt Position (CDP)/debt issuance model, eliminating reliance on external trusted oracles. The system seeks to strike a balance between traditional lending protocols and pure peer-to-peer lending, providing a decentralized and secure environment for savings and credit operations.


## Protocol Arcitecture 


1 . Core Components:

- `Core`: Manages access control and is referenced by all contracts.
- `GuildGovernor`: Governs DAO votes with roles such as DAO_GOVERNOR_GUILD.
- `GaugeVoting`: Allows GUILD holders to vote on lending term allocations.
- `LendingTerm`: Manages individual loan terms, collateral, and interest rates.
- `CreditToken & GuildToken`: ERC-20 tokens representing credit and governance within the system.

2 . Governance Mechanisms:

- `Quorum Actions`: GUILD holders can adjust global debt ceilings, approve new lending terms, and more, subject to quorum and veto.
- `Gauge Voting`: Individuals stake GUILD to vote on lending terms, earning a share of interest paid by borrowers.
- `Borrowing and Liquidation`: Open to anyone, with liquidation occurring through a Dutch auction system.
- `Saving`: Allows users to earn yield on gUSDC through a rebase mechanism.
- `Guild Minting`: Users stake gUSDC to mint GUILD tokens for voting, fostering external capital participation.

3 . Security Measures:

- `Callable Loans`: Eliminates atomic, on-demand withdrawals, preventing bank runs.
- `Bad Debt Handling`: Proportional mark-downs of gUSDC value and slashing mechanisms for stakers in associated gauges.
- `Bootstrapping Credit`: Guarded beta with limited borrowing to test collateral pricing and liquidation systems.
- `Governance Transition`: Emergency powers retained during beta; explicit processes and burning of governance powers post-beta.



## Contract  Overview 

1 . Core.sol


The Core.sol contract is the central access control mechanism for the Ethereum Credit Guild, managing roles and permissions within the ecosystem. It establishes key roles with specific privileges and allows for the dynamic addition and administration of new roles as the protocol grows. Initially, it grants significant control to the deployer, with a design that encourages a transition to a more decentralized governance structure over time. The contract ensures security and transparency by utilizing OpenZeppelin's proven RBAC implementations and is built to support the protocol's self-regulatory functions with minimal external intervention.


2 . CoreRef.sol

The CoreRef contract provides a link to a core contract for role-based access control and includes emergency intervention capabilities. However, it centralizes significant power in the GOVERNOR role, which can change core references and execute arbitrary transactions, posing security risks. The contract also allows pausing and unpausing of functions, which is beneficial for emergency stops.


3 . CoreRoles.sol

The CoreRoles.sol library is a well-structured and cleanly written piece of code that serves as a foundational component for role-based access control within the Ethereum Credit Guild ecosystem. The library defines a comprehensive set of roles, each represented by a bytes32 hash, which are intended to be used by other contracts to manage permissions for various operations.


4 . GuildGovernor.sol

The GuildGovernor contract is a tailored governance solution for the Ethereum Credit Guild, built on OpenZeppelin's governance framework. It features adjustable governance parameters, role-based access control, and a quorum management system. The contract allows for token-based voting, time-delayed execution of proposals, and provides special privileges to Core GOVERNOR and GUARDIAN roles for enhanced security and flexibility in the governance process.

5 . GuildTimelockController.sol

The GuildTimelockController is a custom governance contract that modifies OpenZeppelin's standard TimelockController by centralizing access control through an external Core contract. It bypasses the native role management of AccessControl to maintain consistent permissioning across the ECG system it's part of. The contract minimizes code changes to maintain compatibility with OpenZeppelin's updates and delegates all role management to the Core, indicating a design focused on security and streamlined governance operations.

6 . LendingTermOffboarding.sol

The LendingTermOffboarding contract is a governance tool within a DeFi ecosystem that enables GUILD token holders to democratically remove LendingTerm contracts from the protocol. It facilitates the creation and support of offboarding polls, requires a quorum to pass proposals, and allows for the execution of offboarding, which prevents new loans under the term and removes voting rights for that term. The contract also manages cleanup operations post-offboarding and includes a pause function for emergency stops. Events are emitted for transparency and tracking of the offboarding process.

7 . LendingTermOnboarding.sol

The LendingTermOnboarding contract is a governance-focused factory for creating and managing lending terms in a DeFi setting, utilizing OpenZeppelin's libraries for security and standardization. It integrates governance controls, enforces time-locked proposals, and uses immutable variables for critical references, which are positive features for security and gas efficiency.

8 . ProfitManager.sol

The ProfitManager contract is a core component of a DeFi lending platform, responsible for managing profits and losses, maintaining a surplus buffer for risk mitigation, and adjusting the value of a credit token in response to the platform's financial performance. It enables configurable profit sharing among stakeholders, facilitates the claiming of rewards by users, and is governed by predefined roles to ensure secure and authorized operations. The contract's design aims to ensure transparency, fairness, and resilience in the platform's economic activities.


9 . GuildVetoGovernor.sol

The GuildVetoGovernor contract is a specialized governance tool for the  Ethereum Credit Guild that empowers token holders with the ability to veto scheduled actions in a linked TimelockController. It is built on OpenZeppelin's governance framework but is unique in that it only allows for 'against' votes, focusing on defensive governance. The contract features an adjustable quorum, immediate voting without delay, and an extended voting period governed by the timelock's action timestamp. It prioaritizes security by restricting proposal capabilities and ensuring one vote per token holder. This governance model emphasizes community oversight and control, ensuring that the guild's actions remain in check with the token holders' interests.


10 . ERC20Gauges.sol

The ERC20Gauges contract that integrates gauge-style voting with ERC20 tokens, allowing for resource allocation decisions by token holders. It benefits from using OpenZeppelin's trusted libraries and emits detailed events for transparency. However, the contract has potential issues with the non-retroactive enforcement of the maxGauges limit and lacks explicit reentrancy protection, which could pose risks if not addressed in future iterations. While the contract is generally solid, improvements could be made in handling maxGauges changes, optimizing gas usage, enhancing security, and improving the user experience for managing gauge allocations.

11 . ERC20MultiVotes.sol
The ERC20MultiVotes contract is an abstract Solidity contract that extends the ERC20 token standard to support multi-delegation of voting power. Token holders can delegate their votes to multiple delegates, and the contract keeps track of these delegations through checkpoints, allowing for historical vote tallying. A maxDelegates limit is set to prevent gas-based DoS attacks, but contracts can be authorized to exceed this limit. The contract includes functions for vote delegation, undelegation, and querying vote balances, both current and historical. It also overrides standard ERC20 functions to account for delegated votes and implements EIP-712 for secure off-chain vote delegation. 


12 . ERC20RebaseDistributor.sol

The ERC20RebaseDistributor is an advanced Solidity contract that augments the standard ERC20 token with a rebasing feature, enabling proportional distribution of rewards among token holders who opt into the rebasing mechanism. This contract stands out for its implementation of a share-based system and the use of interpolation to gradually adjust share prices over time, ensuring a smooth distribution process. While the contract offers transparency and incentivizes participation through its reward mechanism, its complexity presents a steep learning curve for users. Additionally, the contract's gas efficiency and potential for initial share price manipulation are areas that could benefit from optimization and preventative measures.


13 . CreditToken.sol

The CreditToken contract is a specialized ERC20 token designed for the Ethereum Credit Guild, featuring additional governance and supply management capabilities. It allows for minting of new tokens by authorized roles, burning to decrease the total supply, and has a unique voting system that supports multiple delegates per voter. The contract also incorporates rebase functionality, which can adjust token balances in response to certain triggers. Access to critical functions is restricted through role-based permissions, ensuring that only designated parties can perform sensitive operations. The contract's behavior is fine-tuned by overriding standard ERC20 functions to integrate its advanced features, such as rebase and multi-vote mechanics.


14 . GuildToken.sol

The GuildToken smart contract serves as the governance token for the Ethereum Credit Guild, with a unique feature set tailored for managing lending terms and associated risks. Upon deployment, the token is non-transferrable, with the exception of minting and burning operations. The contract incorporates a gauge system that allows GUILD holders to vote on lending terms, effectively setting debt ceilings for each term. If a lending term incurs bad debt, the contract has mechanisms to notify and apply losses to the GUILD tokens that voted for the affected gauge, rendering them non-transferable and subject to slashing until the loss is realized.

15 . AuctionHouse.sol

The AuctionHouse contract is part of the Ethereum Credit Guild system, designed to facilitate the auctioning of collateral to recover outstanding CREDIT debt from borrowers. The contract is built upon a CoreRef reference contract, which likely provides shared functionality and access control mechanisms. The auction process is divided into two phases based on a midpoint duration, with the first phase offering an increasing amount of collateral for the full debt amount and the second phase offering all collateral for a decreasing debt amount. The contract includes events to signal the start and end of auctions, functions to start auctions, bid on them, and forgive loans in case of unsuccessful auctions. The startAuction function can only be called by authorized lending terms with the GAUGE_PNL_NOTIFIER role, ensuring that only eligible entities can initiate the auction process. The bid function allows participants to bid on active auctions, with the winning bidder's CREDIT tokens being transferred to the lending term in exchange for the collateral. The forgive function provides a mechanism to mark a loan as a total loss if the auction concludes without any bids, even for zero CREDIT. The contract also maintains a count of ongoing auctions and provides a getter to retrieve auction details.


16 . LendingTerm.sol

The LendingTerm contract is a sophisticated financial instrument within the Ethereum Credit Guild ecosystem, designed to manage the issuance and lifecycle of debt. It facilitates the borrowing of funds by accepting collateral and issuing CREDIT tokens as debt. The contract includes mechanisms for opening loans, adding collateral, making partial repayments, and fully repaying loans. It also features a loan calling process, which allows loans to be called under specific conditions, such as missed repayment deadlines or when the lending term is offboarded.


17 . SimplePSM.sol

The SimplePSM contract is a key component of the Ethereum Credit Guild system, designed to stabilize the value of the CREDIT token in relation to a peg token. It enables users to mint and redeem CREDIT tokens, considering the ProfitManager's creditMultiplier and maintaining transparency through emitted events. The contract's features include pausability of redemptions for governance control. Overall, it contributes to the stability and functionality of the Ethereum Credit Guild ecosystem.

18 . SurplusGuildMinter.sol
The SurplusGuildMinter contract facilitates the minting of GUILD tokens using CREDIT tokens as collateral in a decentralized finance (DeFi) system. Users can stake CREDIT tokens to participate in a gauge voting system, reducing their capital cost and avoiding exposure to GUILD token price fluctuations. Key functionalities include staking, unstaking, and receiving rewards in GUILD and CREDIT tokens. The contract is governed by ratios (mintRatio and rewardRatio), which can be updated by governance. Overall, it plays a vital role in DeFi by providing a mechanism for users to contribute to the surplus buffer and participate in governance activities.


19 . RateLimitedMinter.sol

The RateLimitedMinter contract is  designed to regulate token minting with a rate limit. It inherits from RateLimitedV2 and CoreRef, and includes functions for minting tokens and replenishing a buffer. The contract enforces access control, ensures the contract is not paused, and relies on external contracts for certain operations. It provides a flexible and controlled mechanism for token minting within specified limits.


20 . RateLimitedV2.sol

 RateLimitedV2 Contract is an abstract implementation of a rate-limited mechanism for controlling the frequency of certain actions within a smart contract system. It includes parameters for rate limits, a buffer size, and functions to manage and enforce these limits. The contract allows governance to set and adjust rate limits and provides events for tracking buffer usage and updates to rate limits and buffer caps. Overall, it establishes a flexible framework for implementing rate-limited actions in Ethereum smart contracts.



## Economic Model & Risk Analysis 
1 . ** Economic Model Overview**
The Ethereum Credit Guild (ECG) Protocol introduces a governance token, GUILD, which is central to its economic model. The token is used for voting on governance proposals, managing the protocol's parameters, and participating in the gauge system for debt ceiling management. The gauge system is a key innovation, allowing GUILD holders to vote on lending terms and thereby influence the debt ceilings of various lending products.

The protocol's lending mechanism is designed to facilitate borrowing under specific terms that are voted on and approved by GUILD holders. Borrowers are required to provide collateral, and the terms of loans, including interest rates and duration, are determined by the lending terms associated with each gauge.

The economic incentives are structured to encourage responsible borrowing and timely repayment. Interest rates are set to be competitive with the broader DeFi market, and borrowers are incentivized to maintain healthy collateralization ratios to avoid liquidation.

 **Revenue Streams**
The protocol generates revenue through interest payments on loans and, potentially, fees associated with loan origination, liquidation, or other services. These revenues are distributed to GUILD holders who participate in governance and provide liquidity, aligning their interests with the protocol's success.

2 . **Risk Analysis**
**Credit Risk**

- Borrowers may default on their loans, leading to losses for the protocol. The protocol mitigates this risk by enforcing collateralization ratios and liquidation processes. However, the effectiveness of these measures depends on the volatility of the collateral assets and the efficiency of the liquidation mechanism.

**Market Risk**

- The value of collateral assets can fluctuate significantly, affecting the protocol's ability to recover funds in the event of borrower defaults. The protocol must carefully select acceptable collateral types and set appropriate collateralization ratios to manage this risk.

**Liquidity Risk**

- In times of market stress, the protocol may face challenges in liquidating collateral quickly without causing significant price impact. This risk can be mitigated by maintaining a diversified pool of liquid collateral assets and by integrating with liquid secondary markets.

**Governance Risk**

- The concentration of GUILD tokens among a small number of holders could lead to governance decisions that favor a minority at the expense of the broader community. The protocol should aim for a broad distribution of GUILD tokens and implement mechanisms to prevent governance attacks.

## Centralization Risks

- The quorum requirements for certain governance actions may result in centralization if a small group of large stakeholders can easily achieve the quorum. This could lead to decisions that do not represent the broader community.
- Large stakeholders may dominate gauge voting, potentially centralizing power and decision-making in the protocol.
-  If a significant portion of GUILD tokens is concentrated in the hands of a few entities or individuals, it may lead to centralized 
- Retaining emergency powers during the beta phase could centralize decision-making authority in the hands of a select group.
- Certain addresses with trusted roles, such as DAO timelock, may wield significant power. If these actors are compromised or act maliciously, it could centralize control.
-  Limiting borrowing during the guarded beta phase might concentrate influence and control over the protocol in the hands of a select few.
- The ability for external participants to mint GUILD tokens may centralize influence if a few entities provide significant first-loss capital.





## Systematic Risk 

- Liquidations in the Credit Guild occur after a lending term has been offboarded. While there is no penalty, the text acknowledges the trade-off between auction duration and bad debt risk. The protocol aims to minimize value leakage during liquidations.

- The protocol supports isolated markets by default, meaning bad debt in one market (e.g., ETH) does not affect lenders in another market (e.g., USDC). However, the markets can be linked by accepting each other's deposit receipts as collateral, potentially introducing cross-market risks.
- The protocol aims to minimize trust in certain roles, with plans to "burn" or automate many privileged roles. However, there are still trusted actors, such as those with GOVERNOR and GUARDIAN roles, raising questions about complete trust minimization.
- The gauge system is used to determine the relative debt ceilings of lending terms. GUILD holders collectively decide the relative debt ceilings of all terms, and individual holders can vote in gauges to increase credit limits.
-  The protocol introduces a savings rate for gUSDC holders, allowing them to earn yield through a rebase mechanism. However, it excludes smart contracts like Uniswap pairs that are not compatible with the savings rebase, potentially limiting participation.
- Anyone can stake gUSDC to mint GUILD tokens, which can be used for voting in gauges. This mechanism allows outside participation of first loss capital without exposing the system to potential griefining.


## Learning and Insights 

Having conducted a thorough audit of the Ethereum Credit Guild (ECG) protocol, I've acquired valuable skills that significantly contribute to my expertise in smart contract development and auditing. Here are the skills and insights I've gained

- I now possess a deep understanding of complex decentralized finance (DeFi) systems. The ECG protocol, with its intricate components like governance mechanisms, lending terms, and minting processes, has sharpened my ability to navigate and comprehend intricate smart contract architectures.I now possess a deep understanding of complex decentralized finance (DeFi) systems. The ECG protocol, with its intricate components like governance mechanisms, lending terms, and minting processes, has sharpened my ability to navigate and comprehend intricate smart contract architectures.

- The audit has honed my skills in identifying and mitigating security risks. I am adept at assessing code for potential vulnerabilities and proposing robust solutions to fortify the protocol against malicious exploits. Security considerations are now at the forefront of my development practices.
-  The evaluation of governance models within ECG has equipped me with the knowledge to address challenges related to decentralization. I am skilled in assessing the fairness of decision-making processes and encouraging broader community engagement, ensuring governance aligns with the principles of decentralization.

-  My ability to understand and structure code has significantly improved. I am well-versed in analyzing code organization, the use of inheritance, and the composition of smart contracts. This skill helps me create modular and maintainable code structures.

-  I've developed expertise in testing methodologies and coverage assessments. The use of Forge for unit and integration tests in the ECG audit has ingrained in me the importance of robust testing strategies, ensuring comprehensive coverage to identify and rectify potential bugs.
-  My understanding of community dynamics and governance transitions has grown. I am now well-versed in fostering community engagement and orchestrating a smooth transition of governance powers, crucial for the sustained decentralization of a protocol.



## Recommendation
-  Maintain comprehensive and up-to-date documentation for the entire codebase, including inline comments and external documentation for developers and users.
- Ensure that the governance processes are transparent and decentralized, with clear mechanisms for proposing, voting, and implementing changes.
- Implement circuit breakers or pause functions that can be activated in case of an emergency to halt contract functions while a solution is devised.
- Set up monitoring and alerting systems to track contract activity and flag unusual or suspicious behavior.


## Conclusion


The Ethereum Credit Guild protocol demonstrates a commitment to decentralization, security, and innovation in the DeFi space. As the protocol evolves, addressing recommendations and continuous improvement will be essential to maintaining its integrity and ensuring a resilient and user-friendly financial system on the Ethereum blockchain.










### Time spent:
25 hours