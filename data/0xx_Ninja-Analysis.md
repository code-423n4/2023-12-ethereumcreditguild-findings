## General
I’ve spent 2 days on this audit and covered the code in scope. 
Having being audited twice by private firms, I expected there would to be very little in terms of low-hanging vulnerability
##  Approach
My focus was to understand deeply what Ethereum Credit guild is all about and most importantly how they initiate loan, borrowing and liquidity into protocol.  Guild Credit Protocol introduces a governance-focused decentralized autonomous organization (DAO) with the objective of facilitating lending and borrowing activity. 
The `GuildToken` serves as the native token, offering holders various governance rights. 
The Credit guild incorporates features such as gauge management, loss handling, and transfer restrictions to ensure a secure and efficient operation.

Dedicated over 17 hours during the timeframe of the audit to analyze the codebase. My goal is to provide  with a detailed report that gives them a broader perspective and leverages the value of our research to strengthen the security, usability, and efficiency of the protocol.

## Evaluating codebase
Code quality and documentation is fully developed. The test suite is pretty comprehensive and fuzz tests are a great way to complement the static tests. A vice nice detail to bug is the addition of integration tests to verify behavior of the system as a whole, rather than all its specific sub-components.
## Architecture.

Ethereum Credit Guild implement a mechanism called  **Market Governance**, which enables a diverse range of independent market participants to contribute as  risk bearers.
These engaged lenders then distribute liquidity within the system, earning a premium compared to passive lenders by assuming the risk associated with the loans they endorse. 
The autonomy of participants in selecting their individual risk and yield exposures fosters a more nuanced system, contrasting with one that mandates unanimous agreement on every decision.

For individual GUILD holder, there is a  motivation to promptly divest from loans they have supported if they become precarious to avoid being subjected to slashing, which is comprehensive irrespective of the loss magnitude. 
In instances involving a negligent or dishonest GUILD holder, their counterparts are incentivized to offload the responsibility to enforce slashing, thereby enhancing their own stake and mitigating losses to the surplus buffer.

The core key contracts are;
`GuildToken`: The primary ERC-20 token representing the Ethereum Credit Guild, featuring governance functionalities, gauge management, and multi-voting capabilities.

`CoreRef`: This is reference contract serving as a base component for `GuildToken`.

`ERC20Gauges```: This manage the interaction with gauges, allowing addition and removal, setting maximum gauges, and handling gauge weights.

`ERC20MultiVotes`: Implements multi-voting functionality for purpose of governance.

`LendingTerm`: The contract that relates to lending, providing functions for calculating debt ceilings and ensuring debt management.

`ProfitManager`: Is the centralized contract managing profit distribution accrued by interest after transaction.

## Mechanism review
**Governance**
Governance mechanisms are integrated, allowing key parameters and strategies to be modified by specific roles. The protocol aims to achieve decentralized decision-making through roles like `GUILD_GOVERNANCE_PARAMETERS`.

**Gauge and Loss Mechanism**
They introduces gauges and a loss mechanism, where users are compensated for losses incurred. This  adds a layer of risk management and aligns incentives.
## Systemic Risk

**Centralized Administration Risk**: As the contract has administrator roles that can modify settings or pause the contract, there is a risk that these capabilities may be used improperly or become target for potential attack.

**Price Manipulation Risk**: These contracts are designed to calculate the price of Guild token and creditToken based on price ranges, there is a risk that parameters may be manipulated or that input data may be compromised, potentially leading to incorrect prices.



## Centralization risk
**Access Control Modifiers**: Certain functionalities are restricted to privileged roles, ensuring system integrity but potential for further user control is suggested.

**Withdraw Functionality**: Absence of a withdrawal function limits user control over fund allocation and suggests a centralized approach.

### Time spent:
21 hours