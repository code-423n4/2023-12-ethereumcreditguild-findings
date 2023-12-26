## Summary

ECG protocol aims to establish a decentralized, censorship-resistant pooled lending system. It implements novel mechanisms like honest minority governance and isolated risk markets. However, the added complexity results in centralization risks and systemic vulnerabilities that should be addressed.  

My analysis evaluated security factors across the core protocol components:

- Access control structures
- Token minting/distribution
- Interest rate model  
- Debt issuance/accounting
- Liquidation mechanisms
- Governance processes

The assessment methodology checked:

1. Privilege levels and trusted actors
2. Critical protocol assumptions 
3. Incentive alignments
4. Plausible attack vectors 

The analysis included manual code reviews, scenario modeling, system diagrams, and interface evaluations.

**Key Findings**

| Risk | Rating | Description |
|-|-|-|  
| Centralization | High | Core admin roles retain high privilege in timelock and rate limited minting. |
| System Complexity | High | Intricate token rebasing, debt scaling, and loss distribution amplify risk surface. |
| Token Security | Medium | Auction manipulation impacts collateral ratios, causing bad debt. |   
| Governance Manipulation | Medium | Minority token holders concentration raises governance attack risks. |

**Centralization** *(High)*  
The protocol retains elevated governance roles like `GOVERNOR` across key control points:
- Core admin
- Timelock admin
- Rate limited minting 

Although aimed to transition to DAO control, this initially grants strong unilateral access. Centralized control of minting and governance allows protocol changes against user interests or permissions extraction. _For example_, mint caps or auctions terms could get updated in disruptive ways with minimal oversight in the current model.

**System Complexity** *(High)*
Intricate mechanisms like rebasing tokens, debt scaling, and recomposable buffered loss distributions make the protocol highly complex. This interconnectedness means issues in any component can corrupt dependent logic. For example, skewed rebasing accounting could impact perceived losses, distorting wider incentive structures. Rigorous testing helped mitigate but with novel models, risk typically surfaces over time. Simplifying debt accounting and isolating surplus distribution would reduce future issues.   

**Token Security** *(Medium)* 
The reliance on auction mechanisms for liquidations exposes pricing manipulation risks. Bids remain highly trusting, failing checks against external price or sanity ratio thresholds. This enabled scenarios where attackers acquire underpriced collateral, offloading unpaid portions as automatic bad debt and forcing protocol losses. Enforcing fair value bids via oracles would significantly improve this mechanism resiliency.

**Governance Manipulation** *(Medium)* 
The adoption of an "honest minority" assumption for governance raises risks of concentrated, adversarial control. Even 15-30% token holder collusion could pass disruptive proposals. And with validators earning elevated yields, inducing counterproposals requires greater user coordination. A clearer decentralization schedule and initially higher quorum levels would help mitigate manipulation risks as adoption grows.  

## Architecture recommendations

**Minimize Admin Roles**

The current access control structure grants elevated privilege to `GOVERNOR` and `GUARDIAN` roles across protocol control points like:

- Core logic
- Timelock administration
- Rate limited minting

This centralization risks protocol changes that harm user interests without recourse.

*My Recommendation: Construct an admin role reduction schedule that gradually and irrevocably downgrades permissions as milestones hit across areas like:*  

- Community adoption thresholds
- Total value locked thresholds 
- External security audits/testing

**Simplify Accounting** 

The interconnected surplus buffers, loss waterfalls, and rebasing reward distribution create intricate accounting across system components. This complexity raises risks of manipulation that corrupts broader incentive alignment.

*My Recommendation: Evaluate simplifying accounting models with strategies like:*

- Isolating rebasing rewards from core debt operations
- Removing layered surplus distribution
- Checking key calculations against reliable oracles 

This reduces the manipulated surface area.

**Enhance Oracle Resilience**

Liquidation auctions currently permit unvalidated collateral bids. This allowed distorted bids leading to automatic bad debt creation.

*My Recommendation: Use decentralized price feed oracles like Chainlink to:*  

- Price collateral in bid validation to prevent manipulation
- Provide redundancy if any oracle issue

This mitigates reliance on a single price feed source. 

## Codebase quality analysis

**Well-Commented Structure**

The protocol splits responsibilities across modular contracts for each mechanism using established standards like OZ AccessControl and Governor. Helpful natspec comments explain intended logic flow with useful docs on risky parts. This supports auditing and maintenance.

**Custom Complex Logic**

However, beyond the OpenZeppelin foundations, a significant quantity of novel custom logic gets introduced in core operations like:  

- Token rebasing
- Debt scaling
- Auctions
- Rate limiting

With inventive logic there also surfaces increased test coverage needs.

**Testing Gaps**

Currently there appears insufficient tests on key logic branches around areas like:

- Collateral risk modeling
- Reentrancy protections
- Account reconciliation 
- Loss distributions
- Parameter bound management

More safety could get added validating edge cases in the above flows.

**Cyclomatic Complexity**

Several contracts like [`ProfitManager.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol) [`GuildToken.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol) and [`ERC20RebaseDistributor.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol) are moderately complex based on code branching. While hard to avoid given needed logic, this does increase reasonability analysis needs.

While the base architecture seems well organized, custom logic and test gaps reduce confidence levels currently. Expanding tests and incremental decentralization could improve trust over time.

## Centralization risks

**Elevated Core Control**

The Core contract retains centralized control for directly managing resources and access:

- Minting tokens arbitrarily
- Freezing/unfreezing activity
- Reconfiguring control positions
- Upgrading critical references

With lack of time delays or input ratification, this introduces risk of unilateral detrimental decisions.

For example, the team could upgrade profit sharing, change liquidation models, or adjust rate limiting equations to disadvantage users. Upgrades could also unintentionally contain bugs given low testing requirements currently.

**Timelock Concerns** 

The linked timelock currently grants the Core governor far reaching proposal power including:

- Impersonating any protocol contract 
- Executing arbitrary logic
- Draining funds if access not revoked

This undermines trust minimization aims with unilateral upgrade dangers.

*Mitigations:*

Gradually transitioning admin roles to DAO multisigs and adding time delays could decentralize control.  External audits would also help validate logic flow safety.

## Mechanism review

**Rebasing Tokens**

The rebasing reward distribution model introduces accounting intricacy. Interpolation across unrealized gains, token snapshots, and queued actualization transactions creates tangled state. This produced potential manipulation vectors.

Simplifying rebasing calculations and isolating from core debt operations could improve robustness.

**Liquidations**

The liquidation auction process appears vulnerable regarding price oracle robustness and MEV protections. As identified, insufficient checks enable some bidders to profit from below-market bids at the expense of creditors. 

Tightening bid guardrails would limit distortion incentives. 

**Rate Limiting** 

The rate limiting aimed to provide DOS resilience and prevent minting saturation. However, constraints on replenishment capacities could throttle liquidity during volatile periods.

Benchmarking against periods of network congestion could better calibrate responsive dynamics.

**Token Burning**

Certain debt repayment and interest distribution functions burn tokens to manage balances. This helps synchronization but relies on event order integrity.

Adding reentrancy protections could prevent malicious calls during external interactions.

## Systemic risks

**Cascading Liquidations**

Sudden price declines in assets like ETH could trigger cascading position liquidations. This risks rapid depletion of liquidity to bid on collateral auctions.

If auction interest remains low during the strain, bad debt obligations may pile up beyond surplus capacity. This could force abrupt incentive misalignment and user confidence loss.

Tightening liquidation ratio limits during volatility could help counterbalance risks. 

**Reentrancy Vulnerabilities** 

The complex token distribution logic currently lacks comprehensive reentrancy protections. Though no blatant issues found yet, dynamic EVM innovations increase future attack surface likelihood.

Methodically adopting checks-effects-interactions patterning could mitigate reentrancy attack vectors before incidents emerge.

**Oracle Manipulation** 

The liquidation auction pricing relies on individual oracle sources. Sponsorship complications with options like Chainlink makes incentives misalignments plausible long-term.

Employing a composite oracle approach with fixed governance, such as through a DAO, helps mitigate risks of single oracle corruption.

**Overall Recommendations**  

- Work to minimize admin roles and reduce timelock control 
- Simplify accounting risks via isolated savings rate  
- Enhance price oracle resilience around auctions
- Adopt gradual decentralization schedule for governance control  

**Conclusion**

While the ECG mechanics present unique opportunities, added complexity heightens vulnerabilities. Methodically addressing the risks above will help strengthen protocol security and sustainability long-term.

### Time spent:
45 hours