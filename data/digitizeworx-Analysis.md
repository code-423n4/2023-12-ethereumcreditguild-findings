## Introduction

The Ethereum Credit Guild (ECG) protocol aims to create a decentralized and trust-minimized lending platform for savings and credit. It strives to strike a balance between pure peer-to-peer models and traditional DeFi protocols that rely on trusted price feeds and governance processes. 

The key mechanisms explored in my analysis include:

- Isolated lending markets and terms to contain risk
- First-loss protections via surplus buffers 
- Bad debt markdowns to protect creditor value
- Incentivized participation for governance and stability
- Automated, transparent processes for system integrity

A compartmentalized architecture separates critical functions across modules with explicit capabilities and constraints. This enhances robustness while allowing incremental decentralization over time.

The report evaluates ECG across dimensions like architecture, centralization risks, economic incentives, and systemic risk protections. It also proposes enhancements where applicable, such as an automated debt ceiling mechanism based on on-chain data.

While certain governor privileges remain necessary initially, ECG demonstrates how protocols can minimize trust, attract broad participation, and structurally sustain themselves in turbulent market conditions.

My analysis evaluated the ECG contract architecture, data flows, access controls, and mechanism game theory across the protocol. I created sequence diagrams to analyze user journeys and reviewed natspec comments to fully grasp intended functionality and safety assumptions. Each area of concern or curiosity was explored further with targeted unit test scenarios to probe failure modes and stress behaviors.

**The Architecture** 

ECG employs compartmentalization, separating key risks like isolated lending markets and terms, surplus buffers per market and globally, and discrete governance functions into modular contracts with narrowly scoped capabilities. This enhances robustness.

Further decentralization could be achieved by automating parameters like debt ceilings, minimizing reliance on the Governor role over time. I've outlined a potential automated mechanism below.

The ECG system breaks down key functions into discrete core components:

**Isolated Lending Markets**

Each collateral type (ETH, stablecoins etc.) has its own isolated lending market. This contains risks and losses to each specific market:

- Debt ceilings are allocated per market based on voting 

- Surplus buffers are tracked per market

- Bad debt losses affect each market's credit multiplier separately

So contagion risk is lowered between collateral types.

**Modular Lending Terms**

Within each market, lending terms are deployed that specify distinct borrowing parameters like interest rates, repayment schedules etc.

Terms are isolated from each other - losses in one term do not directly impact others. This prevents risk cascades.

**Targeted Governance Capabilities** 

Critical governance functions are split into modular contracts with explicit capabilities, including:

- Onboarding / offboarding lending terms
- Rate limited token minting
- Profit sharing between stakeholders
- Liquidation auctions

If one governance component is compromised, it minimizes impact on the entire system.

This separation of concerns makes the system more adaptable and resilient compared to a monolithic design. It also follows best practices like the principle of least authority.

**Code & Documentation Quality**

The codebase follows best practices like CEI ordering, use of OpenZeppelin contracts for basic functionality like ERC20s, and ample NatSpec comments explaining rationale and safety assumptions. Areas that could be improved include formalized architectural docs, more aggressively minimizing new code (fork when possible), and additional test coverage.

## Centralization Risks

The Governor and Guardian roles are meant to be transitional but still represent central points of control. As the introductory analysis states, a transition plan could gradually shift from these roles by deploying purpose-specific automated systems for governance functions.

**Necessity of Governor & Guardian Roles**

These powerful roles allow privileged functions like:

- Upgrading contracts
- Changing parameters
- Freezing activity
- Recovering assets

A protocol can't be fully trustless at launch - it requires some agent with heightened permissions to steward initial configurations and emergency interventions.

So ECG intelligently compartmentalizes these dangerous capabilities into specific roles that external holders are meant to control via a time delayed multisig for the initial phases.

**Transitioning Protocol Ownership to Users**

The goal should be to progressively remove ownership dependence - this enhances decentralization and censorship resistance long term. 

One approach is deploying autonomous systems focused on specific governance capabilities according to transparent and predictable rules. For example:

- Automated auctions for liquidations
- Timelocked voting for parameter changes  

User representation can be required in these systems via token weighted voting.

Code transparency and initially conservative but expanding permissions facilitate the confidence needed to gradually shift control into these community owned and operated systems.

## Mechanism Analysis

Incentives around providing first-loss for higher reward ratios using the SurplusGuildMinter seem aligned. Borrower interest rates are fixed per term, avoiding oracle manipulation. Auction design facilitates rapid liquidation. Areas that may need governance overrides are malicious token collateral and uncooperative bids.

**Well-Aligned Staking Incentives**

The [SurplusGuildMinter](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol) allows staking CREDIT for higher governance power and rewards in the form of GUILD tokens. This aligns incentives well:

- CREDIT absorbs first losses, protecting GUILD holders
- More skin in the game earns greater voting power  
- Rewards are variable based on protocol performance

This staking dynamic helps attract first loss protection and rewards active governance.

**Mitigated Oracle Risks**

Borrower interest rates are fixed for each loan's lifetime based on its originating term. This avoids manipulations of an interest rate oracle.

Collateral ratios are also fixed, not dynamically adjusted based on prices.

**Rapid Liquidation Mechanism** 

The sliding scale auctions facilitate rapid liquidations by efficiently price discovering collateral.

**Governance Intervention Needs**

A few cases that could require override are:

- Malicious collateral tokens (e.g. proxy upgrades)
- Flash loan attacks on auctions
- Uncoopensive bidding

Overall the system fundamentals seem sound, but governance needs to remain vigilant against emerging attack vectors on periphery aspects like auctions and collateral options.

**Surplus Guild Minter**  

Users choose their desired tradeoff between risk and reward by staking CREDIT for various lending terms. More risk earns higher GUILD rewards. This aligns incentives well.

By decoupling governance power from capital at risk, the system facilitates broader participation.

**Interest Rates**

Rates are fixed per term based on risk, avoiding manipulative floating rates. As terms are short lived, regular re-pricing occurs.

The system disincentives both irrational borrowing and lending by balancing fixed premiums charged to borrowers with governance powered constrained debt ceilings.

**Auctions**

The sliding scale auctions ensure timely liquidation while keeping the collateral up for auction as long as possible to maximize recovery.

The race dynamic incentivizes rapid arbs. Uncooperative bidding is still a risk, hence the governance override option.

**Governance Intervention**

The overrides are key - while positive incentives are structured to maximize cooperative participant behavior, governance powers like loan forgiveness and emergency shutdown provide the necessary circuit breakers to minimize losses from unpredictable edge cases like malicious ERC20s.

##Systemic Risks

The isolated lending markets, per-term surplus buffers, and global surplus + bad debt markdowns provide defense-in-depth against cascading liquidations.  Explicitly blocking rebase tokens as collateral and limiting collateral risk exposure via weighted debt ceilings also help insulate the system. There does not appear to be a stable peg mechanism for the debt token itself.

**Isolated Markets & Terms**

The compartmentalization of collateral types and discrete lending terms prevents contagion spread. Losses are contained.

**Surplus Buffers** 

The surplus buffers provide the first line of defense against bad debt from market downturns or individual loan losses. This prevents immediate impact on creditors.

**Bad Debt Markdowns**

If losses exhaust the surplus, the global credit multiplier preserves value for creditors by writing down debt obligations. This provides another layer before creditors take losses.

**Conservative Collateral**

Blocking volatile collateral like rebasing tokens reduces black swan risks. The weighted debt ceiling model also limits large directional exposure.

**Potential Debt Token Peg**

While creditors are insulated from losses, the debt token itself doesn't have a pegging mechanism currently. This could undermine longer term confidence in its value. 

Some options are fractional reserve backing similar to real world IOUs, algorithmic supply adjustments a la AMPL, or simply pegging to a major stablecoin like USDC via Uniswap pools.

## Automated Debt Ceiling Mechanism Concept

```solidity
         +----------------------+
         |                      |
         |   Risk Modeling      |
+-------->                      |
         |                      |
         +----------^-----------+
                    |
                    |
                    |
                    |
                    |
                    |
       +------------+-------------+
       |            |             |
   +---+----+   +----+-----+   +-----+----+
   | Market 1 |   | Market 2 |   | Market 3 |
   +---+----+   +----+-----+   +-----+----+
       |            |             |
       |            |             |
       |            |             |
   +---+---+    +---+---+     +---+----+
   | Term 1 |    | Term 1 |     | Term 1 |
   +--------+    +---+---+     +---+----+
                      |
                           +---+----+
                           | Term 2 |
                           +---+----+
```
Higher risk scores lead to lower allocated debt ceilings. As scores improve, debt ceilings expand dynamically. This allows an autonomous, decentralized process for setting prudent ceilings. The Governor just inputs risk model parameters.


### Time spent:
60 hours