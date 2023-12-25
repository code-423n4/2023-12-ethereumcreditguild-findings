
![ECG](https://github.com/code-423n4/2023-10-ethena/assets/125544245/278ef9b0-d13a-4688-9c37-618eb3d4883a)

After thoroughly reviewing the ECG codebase and analyzing potential issues related to malicious collateral, rebasing vulnerabilities, accounting attacks, and auction gaming, here is my comprehensive architectural analysis of the overall protocol.

> The Architecture

The ECG protocol is fundamentally designed to enable decentralized, customizable lending markets without relying on external price oracles. By isolating risk into discrete lending terms while still allowing interoperability, the architecture aims to balance flexibility and stability.

**Isolated Lending Terms**

The [`LendingTerm.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol) contracts create siloed lending pools by collateral type. This compartmentalizes risk and failures. For example, ETH, DAI, and USDC backed loans each have their own pools with tailored parameters like durations, interest rates, collateral ratios, etc.

**Carefully constructed parameters provide appropriate checks and balances between asset volatility and lending risks.**

If one pool develops issues, it can be discontinued without infecting broader liquidity or obligating participants in other markets. Composability between pools still enables using stablecoins from one as collateral in another.

**Customizable Governance** 

The [`GuildGovernor.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol) and [`GuildVetoGovernor.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol) modules enable GUILD stakers to vote on risk parameters and resource allocation between lending pools based on perception of stability.

Rather than institutions making centralized policy decisions, emergent governance allows crowdsourced signaling and feedback. Participants are economically aligned with sustainability.

**Staged Liquidation**

The [`AuctionHouse.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol) provides transparent collateral auctions if borrowers default on loans. This allows market settlement of bad debt without immediate socialization. Participants can monitor and bid based on external price data.

If auctions fail, the `creditMultiplier` value marks down system debt levels proportionately. This balances adverse selection risks.

**Flexible Shutdown** 

The `LendingTermOffboarding` process allows gracefully discontinuing a lending pool through governance if needed. This provides clear procedures around winding down activity.

_The architecture aims to sustain varied risk environments while maintaining continuity of critical monetary services. Custom risk dial setting avoids mandates._

> Core Contracts

The [`Core.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/Core.sol) and [`CoreRef.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol) contracts manage protocol access control and privileges using a role-based model with governor and guardian level supervision. This aims to provide an administrative foundation with emergency capabilities.

The [`CreditToken.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol) and [`GuildToken.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol) implements the CREDIT and GUILD ERC20 tokens with extended logic for rebasing, voting, and loss handling capabilities.

**Access Control**

The `Core` contract maintains roles and permissions using a hierarchical, role-based access control model with governor and guardian oversight. 

Lower privileged agent roles sublicense capabilities around lending actions, surplus handling, etc. This aims to promote least authority patterns.

**Emergency Controls**

The guardian role serves as a protocol protector with pause capabilities, while governor holds ultimate but transient control to enact settlement measures in extremis events.

**Tokenized Rights**

The `GuildToken` and `CreditToken` ERC20 contracts encapsulate governance influence and credit obligations into easily composable and transferable units.

**Rebasing Incentives**

By tracking pro-rata ownership of the credit supply, the `CreditToken` contract allows proportionately distributing accrued interest yields to aligned stakeholders.

**Loss Absorption** 

The `GuildToken` voting weight slashing mechanics penalize support for failed lending pools while retaining continuity and composability.

The tiered administrative structure aims to balance integrity assurances with pragmatic oversight across varied protocol subsystems.

> Lending Terms

The [`LendingTerm.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol) contracts create isolated lending pools where approved ERC20 collateral types can back the minting of CREDIT debt obligations. Critical parameters like interest rates, durations, and collateral ratios are modular to sustain many heterogeneous markets.

The `LendingTerm` contracts are a critical piece of the architecture enabling decentralized, self-governed lending facilities. By modularizing key variables, heterogeneous risk environments can be maintained.

**Isolated Pools**

Each `LendingTerm` creates a siloed market were approved ERC20s can be locked as collateral to issue CREDIT obligations. Segregating by collateral type contains asset volatility.

**Customizable Parameters** 

Terms like interest rates, durations, collateral ratios, and repayment schedules are tailored for the underlying deposited assets.

Conservative stablecoins can sustain higher degrees of leverage with lower rates. More volatile collateral may require overcollateralization.

**Constraint Controls**

Hard caps on debt ceilings limit absolute risks. Attempts to exceed allocated borrowing capacity triggers exponential interest rates to induce equilibrium.

This aims to sustain varied yield opportunities without excessive pooling.

**Maturity Management** 

Partial repayment requirements and auction procedures enable managed growth towards term maturity rather than acute terminations.  

In total, the modular design allows sustaining a wide range of lending environments while avoiding cookie cutter parameters. 

> Governance Modules

The [`GuildGovernor.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol) and [`GuildVetoGovernor.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol) enable GUILD holders to steer protocol parameters and resource allocation at the system level based on voter perceptions of underlying lending term stability.

The governance modules provide formalized mechanisms for decentralized participants to directly guide policy parameters:

**Emergent Interest Rates**

The `GuildGovernor` allows GUILD stakers to vote on collateral-specific variables like interest rates and debt ceilings for lending terms based on risk perceptions.

Rather than institutions setting unilateral policy, rates emerge from crowdsourced signals on asset stability.

**Granular Resource Allocation**

By allocating debt ceilings across isolated lending pools using the gauge weighted voting, the architecture allows targeted redirection of liquidity towards productive lending environments.

Participants have skin-in-the-game around capital efficiency with financial upside for good assessments.  

**Veto Capability**

The `GuildVetoGovernor` enables GUILD holders to cancel pending changes across active timelock administered processes. 

This acts as asafeguard against exploited administrative operations.

In combination, the governance constructs aim to balance efficiency and guidance with protections against manipulation. 

> Risk Management

The [`ProfitManager.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol) facility handles accrued interest flows, surplus buffer administration, and the `creditMultiplier` value used to proportionately manage bad debt impacts on outstanding obligations.

The `ProfitManager` component is critical for balancing risks across varied lending terms with aligned incentive distributions:

**Interest Handling**

As loans accrue yielded interest obligations over time, the `ProfitManager` acts as an accounting facility to track and proportionately allocate gains to stakeholders like GUILD voters supporting productive pools.

**Surplus Administration** 

The surplus buffer allows a capital reserve to absorb losses without immediately impacting end users. The `ProfitManager` contains logic for metapool balancing and governance withdrawals.

**Proportionate Loss Recognition**

By tracking and universally adjusting the `creditMultiplier` parameter against total debts, market losses can be acknowledged without liquidity events. This aligns impact with exposure.   

**Incentive Alignment** 

Mild penalty incentives nudge governance providers towards properly calibrated gauge voting self-assessments on lending stability.

In total, the subsystem aims to dynamically adjust leverage and impacts to sustain varied risk environments while avoiding terminal liquidations. 

> Emergent Pricing

Rather than relying on external oracles for value adjustments, governance permissions around interest rates and collateral ratios combined with the emergent `creditMultiplier` level credibly balance risk across varied lending terms without centralized intervention.

**Collateral Ratios**

Governance voters can signal risk perceptions on lending terms by directing high or low stablecoin debt ceilings against target collateral pools.

Higher debt tolerance implies confidence which translates to efficiency.

**Interest Rates**

Similar to adjusting collateral ratios, GUILD holders can vote to nudge interest rates up or down for assets in a lending term to balance risk and reward.

**Credit Multiplier**

Rather than directly integrating external prices, the architecture allows risk to transparently accumulate across lending pools into the system-wide `creditMultiplier` - diluted value markers on debt obligations.

This proportionately aligns impact without mandatory liquidation events.

**Platform Leverage**

In combination, indirect governance controls on lending conditions and contingent debt value tracking aims to sustain sufficient incentives for varied capital deployment without explicit price data dependencies.

> Stability

In extremis, the [`LendingTermOffboarding.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol) and [`GuildToken.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol) staking slash mechanisms provide transparent wind down pathways to stabilize markets during asset contractions while avoiding socialized losses.

Enabling graceful stability controls during periods of economic contraction rather than cascading liquidations:

**Isolated Discontinuation** 

If specific collateral assets or lending terms begin underperforming, the `LendingTermOffboarding` module provides a pathway for gracefully halting activity without infecting broader liquidity pools.

**Proportionate Penalties** 

The `GuildToken` slashing mechanics apply penalties to GUILD governance providers that supported failed terms, while avoiding platform-wide contagion.

**Market-Driven Timelines**

Rather than relying on discrete liquidation events, the staged `AuctionHouse` combined with dilutive `creditMultiplier` tracking aims to provide market-oriented timelines for settlement based on external buyer signals.

**Recapitalization Avenues** 

The surplus buffer mechanics enable smoother recapitalization avenues compared to abrupt liquidity impacts.

**Modular Wind Downs**

By combining segmented lending facilities with managed control planes around buffer recharging, underperformance is intended to trigger more predictable contractions along market-driven timelines rather than sudden terminal events.

This aims to provide more humane pathways during stability operations compared to institutional mandates around asset continuation.

> Recommendations

1. Formal verification focused on key invariant preservation and reorder attack resilience.

2. Enhance surrogate oracle protections on collateral asset manipulations.  

3. Rigorously isolate state across subsystem boundaries.

4. Implement gradual parameter shifts during economic contractions.

### Time spent:
50 hours