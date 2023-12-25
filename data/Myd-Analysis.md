## Approach

My audit technique analyzes discrete components and then synthesizes findings against architectural whitepapers to gauge systematic risks. I utilize diagrams to map flows and tables to log key mechanisms. Attack vectors are diagrammed to detail risk scenarios. Code markers pinpoint locations tying analysis to implementation.

**Profit Sharing Integrity**

The profit sharing flows spanning [`LendingTerm`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol), [`ProfitManager`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol), [`GuildToken`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol), and [`CreditToken`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol) rely on tightly ordered notification logic. Functions like:

```solidity
LendingTerm._repay()  
ProfitManager.notifyPnL()
```

Must couple closely to ensure interest gets attributed properly across lending terms based on relative weights. Front-running manipulation could violate this priority.

I recommend formal verification of the notification sequences to guarantee fidelity.

**Collateral Token Risks**

The [`LendingTerm`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol) trust of ERC20 collateral could enable violations as malicious tokens drain or deflate value.

Explicit denylisting of collateral is advised. Additionally: 

- Proxy contract checking
- Monitoring of suspicious valuation fluctuations
- Interface adherence validation

This shores up external margins from unintended behavior.

**Access Control**

Reviewing role management across the core contracts is paramount to tuning authority. As governance decentralizes, streamlining and compartmentalizing permissions reduces attack surfaces.

Especially crucial are the deep call paths opened by functions like `CoreRef.emergencyAction`.

## Architecture

ECG employs an intriguing split between pooled neutral collateralized debt and isolated lending terms allowing automated position creation. This balances liquidity breadth with risk siloing.  

However, the interconnectedness also necessitates diligent access control tuning between contracts to prevent dangerous compositions across mechanisms.

At the core, the Ethereum Credit Guild (ECG) employs a split structure:

**Pooled Liquidity**

The [`SimplePSM`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol) aggregates collateral into deep liquidity pools using market operations to maintain redeemability.

This consolidated breadth attracts capital efficiency and usability.

**Isolated Lending** 

Discrete [`LendingTerm`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol) instances provision specialized, automated loan creation per asset with bespoke parameters.

This contains risk subsets to avoid contamination while allowing customization.

**Interlinked Ecosystem**

Critically, the pooled and isolated layers interconnect for contagion transmission:

- `[LendingTerm`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol) debt finances draw from the pooled collateral
- [`SimplePSM`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol) redemption value depends on [`LendingTerm`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol) solvency  

**Emergent Risks**

So while dividing market types segments capital, the contracts integration introduces fragility risks around:

- Cascading liquidity crises
- Complex parameter tuning
- Unmanageable composition explosions

I recommend carefully validating consistency across layers (risk parameters, margin requirements etc.) and streamlining integrations reduce these attack surfaces.

## Code Quality

Overall the codebase displays sound engineering - usage of inheritance, ample natspec comments, and segmentation into logical contracts. Core logic around accounting and distributions remains complex with room for improvement.

## Centralization Risks

The protocol grants strong authority to `GOVERNOR` and `GUARDIAN` roles for contingency control. Manual assembly of auxiliary contracts also centralizes trust. 

**Migrating to an autonomous governance and extension model helps mitigate this long-term.**

The [`Core`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/Core.sol) contract houses the primary `GOVERNOR` and `GUARDIAN` roles that grant expansive authority.

**Governor Powers**

The `GOVERNOR` can:
- Upgrade any protocol contract
- Manipulate parameters across lending markets
- Change `AuctionHouse` and `ProfitManager` rules
- Reconfigure access controls & role permissions

**Guardian Powers** 

The `GUARDIAN` can:
- Pause core logic across system
- Revoke roles and permissions

This hands full systemic oversight to the assigned Admin Keys controlling the roles. The ECG currently depends on the guiding stability of these key holders without decentralized checks.

Additionally, approved new contract assemblies like `LendingTerm` onboarding relies on the current permissioned structure. So loss of keys could inhibit innovation. 

**Over the longer term, migrating admin rights and extension approval flows to autonomous on-chain governance via smart contract votes will help democratize control.**

But for now, the protocol remains fundamentally reliant on specific trusted actors not to abuse their wide powers or lose keys. This necessitates very careful key management in the interim until further maturation.

## Mechanism Review

Innovative designs around ephemeral lending rates and auction liquidations enable neutral open access. However, guaranteeing sequential integrity between rate accrual, profit distribution, and liquidation across contracts is paramount. Formal verification recommended.

While the novel ephemeral lending rate model and auction liquidation design unlock useful properties, ensuring sequential integrity between connected mechanisms is crucial.

The Ethereum Credit Guild (ECG) employs creative ephemeral interest rates tied to market demand signals rather than external data. This allows open participation without centralized oracle risk.

Additionally, the multi-phase collateral auction liquidation avoids permanent position loss.

However, these mechanisms deeply interlink across contracts:

**Lending Term Interactions**

Key interactions include:

- Borrowing Argent signing to open debt position 
- Rate lookups to calculate interest accrual over time
- Collateral auctions to settle positions  

**Integrity Requirements**

Isolated vulnerabilities are insufficient - the end-to-end sequence integrity matters most:

- Interest accumulation visibility before liquidation
- Liquidation triggers reflect accrued balances
- Predictable interest application as debt transitions  

**My Recommendations**  

Formally verifying invariants along cross-contract control flows AP/AR would greatly boost confidence. 

For example, proving interest calculation supplied during auction bid initiation indeed matches signed rate terms at borrowing would lock in reliable sequences.

## Systemic Risks

The interconnected nature of ECG's moving pieces poses overarching model threats. For example, disruptions to the `ProfitManager` accounting flows could corrode reliant downstream debt integrity. Rigorous architectural reviews should pressure test composability & ordering dependence.

At core, ECG demonstrates a deeply interconnected system linking vital functions like:

**Profit Accounting**

Key flows:

- Interest accrual in `LendingTerm`
- Realization by `ProfitManager`
- Distribution to stakeholders

**Debt Integrity** 

Key interactions:

- Loan positions opened by `LendingTerm` 
- System solvency tracked by `ProfitManager`
- Risk parameters and margins set across contracts

**Fragility Risks**

While modularization enables flexibility, linkage introduces fragility:

- Independent changes can cascade unexpectedly
- Breaking core profit sequencing corrodes reliant debt integrity  
- Customizations compound complexity exponentially  

**Stress Testing**

ECG should undergo robust stress testing across layers to validate resilience:

- Fuzzing parameter configurations
- Introducing random delays
- Drama scenarios (e.g. `ProfitManager` hacked)

This provides systemic confidence.

### Time spent:
64 hours