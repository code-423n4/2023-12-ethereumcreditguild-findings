# 🛠️ Analysis - Ethereum Credit Guild Audit
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-12-ethereumcreditguild

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-12-ethereumcreditguild?tab=readme-ov-file#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Ethereum Credit Guild](https://credit-guild.gitbook.io/introduction/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from initial|
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-12-ethereumcreditguild?tab=readme-ov-file#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-12-ethereumcreditguild?tab=readme-ov-file#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-12-ethereumcreditguild?tab=readme-ov-file#scope)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="782" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/e680bd5c-c0f5-4ce4-a9f4-da79018929b3">





</br>
</br>

## Entry Point of Project - Architecture;

<img width="1431" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/8063037f-18b9-48ac-9738-39a6c49cd9a0">

</br>
</br>
</br>
</br>

## The Life Cycle of a Loan - Architecture;

<img width="1247" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/f1c01fd3-1ab9-45d3-9b71-1d96c5653b34">

</br>
</br>
</br>
</br>

## Entry Point of Project - Codes;

![image](https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/5c8c32c7-d77e-402c-ad0a-c94894eda186)


</br>
</br>

## CoreRoles;
I recommend that information about the roles, their authority limitations in the project, and which functions they have which functions should be added to the documents and NatSpec comments;
<img width="960" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/08f60c83-e576-426e-82f4-aa1f81d8dfcc">



</br>
</br>

## Governor Mechanism

<img width="1067" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/f339afe7-765a-4627-b235-0e585236f4b4">
Ref: https://x.com/nmirchev8/status/1736401229377319055?s=20

</br>
</br>

##    <h1 align="center">  test/proposals/gips/GIP_0.sol </h1>


The 'test/proposals/gips/GIP_0.sol' contract within the scope is the distribution file. Why did we start from this project ? Here I would like to offer a different approach to all auditors; To start the code from the right place, examine the distribution file of the project and examine the code according to the distribution there. We will do that now, the chart below shows us how the `test/proposals/gips/GIP_0.sol` distribution file distributes the contracts and in what order.

<img width="1662" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/b5846313-ed66-4102-bfbd-8c4ff8e23e92">


</br>
</br>
</br>
</br>
</br>

## LendingTerm.sol

<img width="1389" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/cedb8bbe-19f4-4453-bcdf-502ad594e9b8">

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>



<img width="1306" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/5a97bc90-f8b7-453a-ad38-27c5d57437b2">

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>




<img width="1597" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/0ef5effd-a68e-4634-aa60-b8ef3f9a317e">


</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

<img width="1630" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/6d90e29d-b2cc-40c7-9c23-6675975a889b">

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>


<img width="1616" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/a43fad0e-a66d-42f5-a92d-09de5af72304">


</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

<img width="1618" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/8da6d5a1-8c84-4c8f-9203-b8fb08be4ec2">

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

#### Main invariants
- Debt value: The debt value should be constant in USDC term (+interest over time), even if the gUSDC token loses value due to bad debt creation.
- Profit distribution: Interest paid by borrowers should be entirely redistributed between lenders (gUSDC holders, current or future due to interpolation of rewards), GUILD stakers (+ GUILD borrowers that stake through the SurplusGuildMinter), and the surplus buffers.
- Issuance: Issuance of lending terms should be 0 if there are no loans open, so that the terms can be offboarded properly.
- Issuances: The sum of all terms issuance should be less than the total debt owed by borrowers (due to interests accruing + eventual loss of values of the gUSDC token that marks up all open debts)

<img width="1585" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/461e03cd-6ba0-4050-8080-b03a4c12b932">


</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

Anyone can repay anyone's debt at any time when the loan is repaid, the principal is burnt, part of the interest is paid to those those GUILD holders staking on that loan's terms, part to the surplus buffer, and part to the gUSDC savings rate if a loan missed a period repayment, anyone can call it (resulting in the liquidation of collateral to pay the debt)

The borrower can bid on their own collateral in the auction to repay their loan, before someone else takes the arbitrage (leaking part of the loan collateral to bidder instead of borrower)


<img width="1584" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/4c347818-24bd-466a-8d18-089816bbbc38">

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

<img width="1596" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/c10c1aaf-fc7b-4b02-9cfa-2b693554151e">



</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. In particular, invariant tests have been written successfully.

-   2) Overall line coverage percentage provided by your tests : 99




### What could they have done better?


-  1) In order to understand the test scenarios and develop more effective test scenarios, the following bob, alice and other roles are can be defined one by one, in this way role definitions increase the quality and readability in tests

```solidity

 // Sample labels
vm.label(bob, 'bob');
vm.label(alice, 'alice');
vm.label(DEPLOYER, 'deployer');
vm.label(USD_OWNER, 'usd owner');
vm.label(POOL_PROXY, 'lending pool');
```
</br>

-  2) Test suites do not test for re-entrancy Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. 
https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability
The accuracy of the functions has been tested, but it has not been tested whether the nonReentrant modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.

</br>

-  3) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="780" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/16aea122-8fbc-4cb4-88db-d3765e5e2498">

Ref:https://xin-xia.github.io/publication/icse194.pdf

</br>
</br>

-  4) 'LendingTerm.sol' is the most important contract of the audit. Therefore, it may be necessary to make additions to the test scenarios in this file; Here you can see the list of available test cases and my additional suggested scenarios.

</br>

## Ethereum Credit Guild  - LendingTerm.sol Test  Analysis
| Test Function                                 | Tested Function                          | Description                                                                                                      | Test Type                            | Test Conclusion |
|-----------------------------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------|--------------------------------------|-----------------|
| testInitialState                              | Various                                  | Verifies initial state of various contract components, including token addresses and lending terms.              | State Verification                   | ✅               |
| testInitialize                                | initialize                               | Checks the proper initialization of a LendingTerm contract and its clone, ensuring correct setup.                | Initialization Success/Failure       | ✅               |
| testBorrowSuccess                             | borrow                                   | Tests successful borrowing, ensuring correct balance updates and loan creation.                                  | Borrowing Scenario (Success)         | ✅               |
| testBorrowWithOpeningFeeSuccess               | borrow                                   | Tests borrowing with an opening fee, verifying fee application and loan details.                                 | Borrowing Scenario with Fee          | ✅               |
| testBorrowFailNoCollateral                    | borrow                                   | Ensures the contract reverts when attempting to borrow without collateral.                                       | Borrowing Scenario (Failure)         | ✅               |
| testBorrowFailAmountTooSmall                  | borrow                                   | Checks if borrowing fails when the amount is too small, expecting a revert.                                      | Borrowing Scenario (Failure)         | ✅               |
| testBorrowFailNoDebt                          | borrow                                   | Checks failure when attempting to borrow with zero debt amount.                                                  | Failure Scenario (Zero Debt)         | ✅               |
| testBorrowFailExists                          | borrow                                   | Verifies failure when a loan already exists for the same borrower.                                               | Failure Scenario (Existing Loan)     | ✅               |
| testBorrowFailNotEnoughCollateral             | borrow                                   | Ensures failure when the collateral is insufficient for the borrow amount.                                       | Failure Scenario (Insufficient Collateral) | ✅               |
| testBorrowFailGaugeKilled                     | borrow                                   | Tests failure when the gauge is killed and borrowing is attempted.                                               | Failure Scenario (Gauge Killed)      | ✅               |
| testBorrowFailRoleRevoked                     | borrow                                   | Checks failure when the rate-limited minter role is revoked.                                                     | Failure Scenario (Role Revoked)      | ✅               |
| testBorrowFailDebtCeilingReached              | borrow                                   | Verifies failure when the debt ceiling is reached.                                                               | Failure Scenario (Debt Ceiling)      | ✅               |
| testBorrowFailHardcapReached                  | borrow                                   | Tests failure when the hardcap is reached.                                                                       | Failure Scenario (Hardcap Reached)   | ✅               |
| testBorrowFailPaused                          | borrow                                   | Ensures failure when borrowing is attempted while the contract is paused.                                        | Failure Scenario (Paused)            | ✅               |
| testBorrowFuzz                                | borrow                                   | Tests borrow functionality under various conditions using fuzz testing.                                          | Fuzz Testing                         | ✅               |
| testAddCollateralSuccess                      | addCollateral                            | Checks success when adding collateral to an existing loan.                                                       | Success Scenario                     | ✅               |
| testAddCollateralFailures                     | addCollateral                            | Tests various failure scenarios for adding collateral.                                                           | Failure Scenarios                    | ✅               |
| testPartialRepaySuccess                       | partialRepay                             | Verifies successful partial repayment of a loan.                                                                 | Success Scenario                     | ✅               |
| testPartialRepayReverts                       | partialRepay                             | Tests various failure scenarios for partial repayment.                                                           | Failure Scenarios                    | ✅               |
| testRepaySuccess                              | repay                                    | Checks successful full repayment of a loan.                                                                      | Success Scenario                     | ✅               |
| testRepayFailLoanNotFound                     | repay                                    | Verifies failure when trying to repay a non-existent loan.                                                       | Failure Scenario (Loan Not Found)    | ✅               |
| testRepayFailCreatedSameBlock                 | repay                                    | Tests failure when attempting to repay a loan created in the same block.                                         | Failure Scenario (Same Block)        | ✅               |
| testRepayFailAlreadyClosed1                   | repay                                    | Ensures failure when trying to repay an already closed loan.                                                     | Failure Scenario (Already Closed)    | ✅               |
| testRepayFailRoleRevoked                      | repay                                    | Tests failure when attempting to repay after revoking the minter role.                                           | Failure Scenario (Role Revoked)      | ✅               |
| testCallSuccess                               | call                                     | Checks successful call action on a loan.                                                                         | Success Scenario                     | ✅               |
| testCallManySuccess                           | callMany                                 | Tests successful call action on multiple loans simultaneously.                                                   | Success Scenario                     | ✅               |
| testCallFailConditionsNotMet                  | call                                     | Verifies failure when call conditions are not met.                                                               | Failure Scenario                     | ✅               |
| testCallFailLoanNotFound                      | call                                     | Ensures failure when trying to call a non-existent loan.                                                         | Failure Scenario (Loan Not Found)    | ✅               |
| testCallFailCreatedSameBlock                  | call                                     | Tests failure when calling a loan created in the same block.                                                     | Failure Scenario (Same Block)        | ✅               |
| testCallFailAlreadyCalled                     | call                                     | Checks failure when trying to call an already called loan.                                                       | Failure Scenario (Already Called)    | ✅               |
| testCallFailLoanClosed                        | call                                     | Verifies failure when calling a closed loan.                                                                     | Failure Scenario (Loan Closed)       | ✅               |
| testCallAfterPartialRepayDelay                | call                                     | Tests the ability to call a loan after a delay in partial repayment.                                             | Success Scenario                     | ✅               |
| testForgiveFailLoanNotFound                   | forgive                                  | Checks failure when attempting to forgive a non-existent loan.                                                   | Failure Scenario (Loan Not Found)    | ✅               |
| testGovernorSetAuctionHouse                   | setAuctionHouse                          | Ensures that only the governor can set the auction house.                                                        | Authorization                        | ✅               |
| testSetHardCap                                | setHardCap                               | Checks if the hard cap is set correctly and handles unauthorized attempts.                                       | Functionality & Security             | ✅               |
| testFlowBorrowRepay                           | borrow, repay                            | Full workflow test for the borrowing and repaying process.                                                       | Workflow                             | ✅               |
| testFlowBorrowCallOnBidGoodDebt               | borrow, call, onBid                      | Tests the entire flow of borrowing, calling, and bidding on a loan with good debt.                               | Workflow                             | ✅               |
| testFlowBorrowCallOnBidBadDebt                | borrow, call, onBid                      | Evaluates the full process of borrowing, calling, and bidding on a loan with bad debt.                           | Workflow                             | ✅               |
| testFlowBorrowForgive                         | borrow, forgive                          | Tests the complete flow of borrowing and forgiving a loan.                                                       | Workflow                             | ✅               |
| testActiveLoansAreMarkedUpWhenCreditLoseValue | markUpOnCreditLoss                       | Tests if active loans are marked up when CREDIT loses value.                                                     | Functionality                        | ✅               |
| testCanBorrowMoreAfterCreditLoseValue         | borrowMoreOnCreditLoss                   | Verifies the ability to borrow more when CREDIT loses value.                                                     | Functionality                        | ✅               |
| testCannotUpdateAfterCall                     | restrictUpdatesAfterCall                 | Checks if updates are restricted after a loan call.                                                              | Restriction                          | ✅               |
| testProfitAccountingRepayAfterMarkUp          | profitAccountingOnRepay                  | Tests profit accounting on loan repayment after a markup.                                                        | Accounting                           | ✅               |
| testProfitAccountingPartialRepayAfterMarkUp   | profitAccountingOnPartialRepay           | Verifies profit accounting on partial loan repayment after a markup.                                             | Accounting                           | ✅               |
| testProfitAccountingCallBidAfterMarkUp        | profitAccountingOnCallBid                | Assesses profit accounting on call and bid after a markup.                                                       | Accounting                           | ✅               |
| testProfitAccountingForgiveAfterMarkUp        | profitAccountingOnForgive                | Evaluates profit accounting on loan forgiveness after a markup.                                                  | Accounting                           | ✅               |
| testMinBorrowAfterCreditLoseValue1            | minBorrowIncreaseOnCreditLoss - borrow() | Checks if MIN_BORROW increases when CREDIT value decreases - for borrow() function.                              | Value Check                          | ✅               |
| testMinBorrowAfterCreditLoseValue             | minBorrowIncreaseOnCreditLoss - partialRepay() | Verifies if MIN_BORROW increases when CREDIT value decreases - for partialRepay() function.                    | Value Check                          | ✅               |


</br>
</br>
</br>
</br>
</br>





## Ethereum Credit Guild  -  LendingTerm.sol additional tests that may be added 

| Test Function                                      | Description                                                                                                  |
|----------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| testBorrowExceedingCollateral                      | Test if borrowing an amount exceeding collateral limit results in a contract revert.                         |
| testBorrowWithMaximumCollateral                    | Check borrowing behavior when the collateral amount is at its maximum limit.                                 |
| testBorrowWithMinimumCollateral                    | Verify if minimum collateral requirements are strictly enforced during borrowing.                            |
| testAddCollateralToNonExistentLoan                 | Test adding collateral to a non-existent loan to ensure it fails as expected.                                |
| testAddCollateralExceedingLimits                   | Check behavior when adding collateral that exceeds predefined limits.                                        |
| testAddCollateralPostLoanClosure                   | Verify that adding collateral is not possible after a loan is closed.                                        |
| testPartialRepayWithAmountGreaterThanLoanDebt      | Test behavior when a partial repayment amount is greater than the current loan debt.                         |
| testPartialRepayWithZeroDebt                       | Ensure that attempting a partial repayment with zero debt results in a failure.                              |
| testPartialRepayBelowMinimumPercent                | Verify that partial repayment below the minimum required percentage is rejected.                             |
| testPartialRepayAtBlockCreationTime                | Test if a loan can be partially repaid in the same block it was created, expecting failure.                  |
| testPartialRepayWithNonExistentLoan                | Ensure partial repayment fails when attempted on a non-existent loan.                                        |
| testPartialRepayWithClosedLoan                     | Check behavior when trying to partially repay a loan that has already been closed.                           |
| testPartialRepayAfterLoanCall                      | Test the ability to partially repay a loan after it has been called.                                         |
| testAddCollateralAndPartialRepayIntegration        | Test the integration and interaction between adding collateral and partial repayment on the same loan.      |
| testRepayWithDifferentCreditMultipliers            | Assess how repayment behaves with varying credit multipliers, especially in edge cases.                      |
| testPartialRepayWithInterestVariations             | Test partial repayments under varying interest rate conditions, including high and low rates.                |
| testBorrowAndRepayFullCycle                        | Test a full borrowing and repayment cycle, including all possible edge cases.                                |
| testAddCollateralEffectOnLoanHealth                | Assess the impact of adding collateral on the overall health and risk profile of a loan.                     |
| testPartialRepayImpactOnLoanDuration               | Examine how partial repayments affect the duration and terms of a loan.                                      |
| testBorrowWithDynamicInterestRates                 | Test borrowing behavior under dynamically changing interest rates.                                           |
| testRepayScenariosWithDifferentRepaymentSizes      | Test various repayment scenarios with different sizes and frequencies of repayments.                         |

</br>
</br>
</br>
</br>

-  5) Test Tools/Technologies analysis of the project;

<img width="983" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7b64505d-9130-4923-a86d-5a33c5ebb9ae">


</br>
</br>
</br>
</br>


## d) Security Approach of the Project

### Successful current security understanding of the project;

1- They manage the 1nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.

7- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

8- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

9- We also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

10 - As the project team, you can consider applying the multi-stage audit model.

<img width="498" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7ad49e46-4998-4bf2-830e-711039ea97a8">

Read more about the MPA model;
https://mpa.solodit.xyz/

11 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/bot-report.md


</br>
</br>

##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)                         | A lot of OZ contracts |-  Version `4.9.3` is used by the project, it is recommended to use the newest version `5.0.1`                                                                                                                  |
| [`solmate`](https://www.npmjs.com/package/solmate)                                                                                        | [![npm](https://img.shields.io/npm/v/solmate.svg)](https://www.npmjs.com/package/solmate)   | SafeCastLib.sol                                                                                 |  - The latest updated version is used. |


</br>
</br>




## g) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
100% comprehensive security cannot be achieved based on smart contract codes alone.
Implement a more comprehensive policy to enforce non-SMS 2FA.
You can find the latest Simswap attack on Code4rena and details about it in this article: 
https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d


✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 


✅ Analyze the gas usage of each function under various conditions in tests to ensure efficiency and identify potential optimizations.	

✅ Especially in mixed Defi functions, it will be easier to read and control the code if NatSpec comments are designed like the example below. The following are examples of mixed Defi protocols. Remember; We are evolving towards a world where visuality is at the forefront.

Here are some of them;

```solidity
Credit ; Panoptic project
------------------------

    /** 
        We're tracking the amount of net and removed liquidity for the specific region:

             net amount    
           received minted  
          ▲ for isLong=0     amount           
          │                 moved out      actual amount 
          │  ┌────┐-T      due isLong=1   in the UniswapV3Pool 
          │  │    │          mints      
          │  │    │                        ┌────┐-(T-R)  
          │  │    │         ┌────┐-R       │    │          
          │  │    │         │    │         │    │     
          └──┴────┴─────────┴────┴─────────┴────┴──────►                     
             total=T       removed=R      net=(T-R)


     *       removed liquidity r          net liquidity N=(T-R)
     * |<------- 128 bits ------->|<------- 128 bits ------->|
     * |<---------------------- 256 bits ------------------->|
     */



    ///   ┌───────────────────────┐
    ///   │mintTokenizedPosition()├───┐
    ///   └───────────────────────┘   │
    ///                               │
    ///   ┌───────────────────────┐   │   ┌───────────────────────────────┐
    ///   │burnTokenizedPosition()├───────► _validateAndForwardToAMM(...) ├─ (...) --> (mint/burn in AMM)
    ///   └───────────────────────┘       └───────────────────────────────┘



/// @notice When a position is minted or burnt in-the-money (ITM) we are *not* 100% token0 or 100% token1: we have a mix of both tokens.
    /// @notice The swapping for ITM options is needed because only one of the tokens are "borrowed" by a user to create the position.
    /// @notice This is an ITM situation below (price within the range of the chunk):
    ///
    ///        AMM       strike
    ///     liquidity   price tick
    ///        ▲           │
    ///        │       ┌───▼───┐
    ///        │       │       │liquidity chunk
    ///        │ ┌─────┴─▲─────┴─────┐
    ///        │ │       │           │
    ///        │ │       :           │
    ///        │ │       :           │
    ///        │ │       :           │
    ///        └─┴───────▲───────────┴─► price
    ///                  │
    ///             current price
    ///           
```
</br>
</br>



✅ The 'test/proposals/gips/GIP_0.sol' contract within the scope is the distribution file. There are `deploy()` and `afterDeploy()` functions that need to be run after that. The deployment process outlined in the deploy() function sets up various components of a protocol, including the core system, profit manager, tokens, minters, auction house, governance mechanisms, and more. It's a complex, multi-step process that initializes different contracts and sets up relationships between them. This is followed by the afterDeploy() function which grants roles and permissions to various actors within the system, finalizes configurations, and performs clean-up tasks like renouncing the deployer roles.

If these functions are not run atomically—in other words, if there is a gap between the execution of deploy() and afterDeploy()—several issues can arise:

- Role Assignment Gap: There would be a period where the necessary roles and permissions are not assigned. This could lead to a window where the protocol is not fully operational or secure, as some components may require specific permissions to function correctly.

- Governance Risk: If the afterDeploy() function is forgotten or delayed, the governance setup would be incomplete. This could prevent the protocol from being governed as intended, potentially leaving it without the ability to make important decisions or respond to issues.

- Security Vulnerabilities: The delay or omission of afterDeploy() could introduce security vulnerabilities. For example, if the roles that manage the governance parameters are not assigned, an attacker could potentially exploit this gap.

The roles related to governance (e.g., TIMELOCK_PROPOSER, TIMELOCK_EXECUTOR) are particularly sensitive. If these are not immediately secured, an attacker could try to grant themselves these roles and take over the governance process.

The afterDeploy() function grants the CREDIT_MINTER and GUILD_MINTER roles to specific contracts. If these roles are not set, an attacker might be able to call functions related to token minting, potentially allowing them to mint tokens to their address, which could lead to theft or inflation of the token supply.

</br>
</br>
</br>


✅ The smart contracts of the project use 3 different SPDX Licenses,

The use of different Software Package Data Exchange (SPDX) licenses in a single Ethereum smart contract project can raise several security and compatibility concerns.

The main licenses in project, GPL-3.0-or-later, AGPL-3.0-only, and MIT. GPL and AGPL licenses are known for their "copyleft" nature, meaning that any derivative work must also be distributed under the same license. The MIT license, on the other hand, is permissive and allows for greater freedom in how the code is used and redistributed. Combining these licenses can lead to compatibility issues, where it's unclear under which terms the combined work should be distributed.

</br>
</br>
</br>


✅ This listing makes imports more understandable;

```diff
LendingTerm.sol

- import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
- import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
- import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";
- import {CoreRef} from "@src/core/CoreRef.sol";
- import {CoreRoles} from "@src/core/CoreRoles.sol";
- import {GuildToken} from "@src/tokens/GuildToken.sol";
- import {CreditToken} from "@src/tokens/CreditToken.sol";
- import {AuctionHouse} from "@src/loan/AuctionHouse.sol";
- import {ProfitManager} from "@src/governance/ProfitManager.sol";
- import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";


+ // OpenZeppelin Contracts
+ import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
+ import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
+ import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

+ // Core Components
+ import {CoreRef} from "@src/core/CoreRef.sol";
+ import {CoreRoles} from "@src/core/CoreRoles.sol";

+ // Token Contracts
+ import {GuildToken} from "@src/tokens/GuildToken.sol";
+ import {CreditToken} from "@src/tokens/CreditToken.sol";

+ // Loan and Auction Contracts
+ import {AuctionHouse} from "@src/loan/AuctionHouse.sol";

+ // Governance
+ import {ProfitManager} from "@src/governance/ProfitManager.sol";

+ // Rate Limiting
+ import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

```

</br>
</br>
</br>
</br>

✅  I recommend that you classify functions such as Public, External, Internal as follows, this is the most effective method to classify functions ; 
<img width="467" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/f6661c98-7595-4ae2-bc5b-b086dc4f3018">

https://x.com/PaulRBerg/status/1657098652987318292?s=20

</br>
</br>
</br>
</br>

✅ `LendingTerm.sol` contract is very critical for project, we see that the `whenNotPaused` modifier is used only in the `borrow` function, the use of this modifier in several other functions will increase security;

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




</br>
</br>
</br>
</br>
</br>
</br>

## h) New insights and learning from this audit 


🔎 1. Token System: The protocol features two types of tokens: GUILD (governance token) and gUSDC (credit token pegged to USDC). While multiple credit tokens are planned for future markets, GUILD remains constant across all markets.

🔎 2. Governance and Quorum Actions: Governance actions, such as adjusting global debt ceilings or approving new LendingTerms, require a quorum of GUILD holders. These actions encompass a range of system modifications and are subject to veto, with distinct quorums and voting periods for each action.

🔎 3. Gauge Voting and Risk: GUILD holders can vote in the gauges for a LendingTerm, influencing its credit limit and exposing their tokens to risk in case of bad debt. Staked GUILD holders earn a portion of the interest, and the ability to unstake is conditional, ensuring accountability.

🔎 4. Borrowing, Liquidation, and Savings Mechanism: Users can mint and borrow gUSDC by providing collateral, with mechanisms for loan repayment and liquidation. The gUSDC savings rate offers yield through a rebase mechanism, and loan liquidation can trigger auctions for collateral. Losses exceeding the surplus buffer lead to a markdown in gUSDC value.

🔎 5. GUILD Minting and Handling Bad Debt: GUILD tokens can be minted by staking gUSDC, providing first-loss capital. The system handles bad debt through callable loans and auction-based liquidation, aiming for fairer debt management compared to traditional DeFi protocols. Governance, during the initial beta phase, retains emergency powers, transitioning to a more decentralized model post-beta.

🔎 6. Additional Context: 
<img width="1476" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/27b3b3c0-2ca1-4a12-a016-7f7ef0085b3f">

### Time spent:
22 hours