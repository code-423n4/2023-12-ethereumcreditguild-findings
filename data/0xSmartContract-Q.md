##### QA Report Issues List

- [x] **Low 01** → `deploy()` and `afterDeploy()` functions contain serious security vulnerabilities because they are not run atomicly.
- [x] **Low 02** → Using 3 different SPDX Licenses may cause compatibility issues
- [x] **Low 03** → Apply the whenNotPaused modifier to other important functions in the LendingTerm.sol contract



### [Low-1] `deploy()` and `afterDeploy()` functions contain serious security vulnerabilities because they are not run atomicly.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/test/proposals/gips/GIP_0.sol#L108C1-L109C1

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/test/proposals/gips/GIP_0.sol#L316


The deployment process outlined in the `deploy()` function sets up various components of a protocol, including the core system, profit manager, tokens, minters, auction house, governance mechanisms, and more. It's a complex, multi-step process that initializes different contracts and sets up relationships between them. This is followed by the `afterDeploy()` function which grants roles and permissions to various actors within the system, finalizes configurations, and performs clean-up tasks like renouncing the deployer roles.
If these functions are not run atomically—in other words, if there is a gap between the execution of `deploy()` and `afterDeploy()`—several issues can arise:

Role Assignment Gap: There would be a period where the necessary roles and permissions are not assigned. This could lead to a window where the protocol is not fully operational or secure, as some components may require specific permissions to function correctly.

Governance Risk: If the `afterDeploy()` function is forgotten or delayed, the governance setup would be incomplete. This could prevent the protocol from being governed as intended, potentially leaving it without the ability to make important decisions or respond to issues.

Security Vulnerabilities: The delay or omission of `afterDeploy()` could introduce security vulnerabilities. For example, if the roles that manage the governance parameters are not assigned, an attacker could potentially exploit this gap.


The roles related to governance (e.g., TIMELOCK_PROPOSER, TIMELOCK_EXECUTOR) are particularly sensitive. If these are not immediately secured, an attacker could try to grant themselves these roles and take over the governance process.

The afterDeploy() function grants the CREDIT_MINTER and GUILD_MINTER roles to specific contracts. If these roles are not set, an attacker might be able to call functions related to token minting, potentially allowing them to mint tokens to their address, which could lead to theft or inflation of the token supply.



### [Low-2] Using 3 different SPDX Licenses may cause compatibility issues

The smart contracts of the project use 3 different SPDX Licenses,

The use of different Software Package Data Exchange (SPDX) licenses in a single Ethereum smart contract project can raise several security and compatibility concerns.

The main licenses in project, GPL-3.0-or-later, AGPL-3.0-only, and MIT. GPL and AGPL licenses are known for their "copyleft" nature, meaning that any derivative work must also be distributed under the same license. The MIT license, on the other hand, is permissive and allows for greater freedom in how the code is used and redistributed. Combining these licenses can lead to compatibility issues, where it's unclear under which terms the combined work should be distributed.



### [Low-3] Apply the whenNotPaused modifier to other important functions in the LendingTerm.sol contract


`LendingTerm.sol` contract is very critical for project, we see that the `whenNotPaused` modifier is used only in the `borrow` function, the use of this modifier in several other functions will increase security;

|  Contract  |         Type        |       Bases      |                  |                 |
|:----------:|:-------------------:|:----------------:|:----------------:|:---------------:|
|     └      |  **Function Name**  |  **Visibility**  |  **Mutability**  |  **Modifiers**  |
||||||
| **LendingTerm** | Implementation | CoreRef |||
| └ | <Constructor> | Public ❗️ | 🛑  | CoreRef |
| └ | initialize | External ❗️ | 🛑  |NO❗️ |
| └ | getReferences | External ❗️ |   |NO❗️ |
| └ | getParameters | External ❗️ |   |NO❗️ |
| └ | collateralToken | External ❗️ |   |NO❗️ |
| └ | getLoan | External ❗️ |   |NO❗️ |
| └ | getLoanDebt | Public ❗️ |   |NO❗️ |
| └ | partialRepayDelayPassed | Public ❗️ |   |NO❗️ |
| └ | debtCeiling | Public ❗️ |   |NO❗️ |
| └ | debtCeiling | External ❗️ |   |NO❗️ |
| └ | _borrow | Internal 🔒 | 🛑  | |
| └ | borrow | External ❗️ | 🛑  | whenNotPaused |
| └ | _addCollateral | Internal 🔒 | 🛑  | |
| └ | addCollateral | External ❗️ | 🛑  |NO❗️ |
| └ | _partialRepay | Internal 🔒 | 🛑  | |
| └ | partialRepay | External ❗️ | 🛑  |NO❗️ |
| └ | _repay | Internal 🔒 | 🛑  | |
| └ | repay | External ❗️ | 🛑  |NO❗️ |
| └ | _call | Internal 🔒 | 🛑  | |
| └ | call | External ❗️ | 🛑  |NO❗️ |
| └ | callMany | Public ❗️ | 🛑  |NO❗️ |
| └ | forgive | External ❗️ | 🛑  | onlyCoreRole |
| └ | onBid | External ❗️ | 🛑  |NO❗️ |
| └ | setAuctionHouse | External ❗️ | 🛑  | onlyCoreRole |
| └ | setHardCap | External ❗️ | 🛑  | onlyCoreRole |


whenNotPause  modifier  can be added to functions? and why we should be add;

- Function: `addCollateral`
Reason for Inclusion: Adding collateral is a sensitive operation affecting the loan's security. In scenarios where the contract's stability is questionable, pausing this function can prevent potentially hazardous changes to the collateral pool.
Risk Mitigation: Prevents the addition of collateral during system anomalies, protecting both the platform and users from unforeseen vulnerabilities.

- Function: `partialRepay`
Reason for Inclusion: Loan repayments impact the platform's financial state. Pausing this during abnormal activities ensures that repayments are processed under stable and secure conditions.
Risk Mitigation: Safeguards against malicious activities or errors during the repayment process, maintaining the integrity of the loan and repayment terms.

- Function: `call` and `callMany`
Reason for Inclusion: These functions likely involve complex interactions with other contracts or perform significant state changes. Pausing them ensures that such interactions only occur in a stable environment.
Risk Mitigation: Prevents potentially exploitable interactions or cascading failures in the event of a bug or attack on connected functionalities.

- Function: `setAuctionHouse and setHardCap`
Reason for Inclusion: These administrative functions change crucial parameters of the platform. Pausing them ensures that no abrupt or unauthorized changes are made during critical periods.
Risk Mitigation: Protects the platform from sudden governance attacks or unexpected changes in operational parameters, ensuring stability and user trust.
