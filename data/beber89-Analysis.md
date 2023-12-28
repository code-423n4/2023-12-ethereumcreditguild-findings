
# Ethereum Credit Guild Analysis

## Description:

The Ethereum Credit Guild aims to find a balance between systems where most people are honest and direct person-to-person lending. 
It uses rewards and safeguards to enable savings and loans without needing to trust a central authority. 
The system can adapt to market changes through an open process. 
It assumes "the minority" is honest, and if you have enough tokens to be part of the honest minority group then
you don't need a centralised party. 
If something goes wrong, the system will stop lending via the lending terms (a loan) and  compensate borrowers. 
ECG brings a unique liquidation mechanism for unhealthy borrow positions (a borrow position is essentially a loan) , 
if the borrower skips a partial repayment then the collateral in the position is auctioned off where bidders can 
bid on the collateral. This is how the "market price" is decided. 
The GUILD is the governance token which is used to decide on decisions for the protocol.


# 1. System Overview

## 1.1 Scoping Details

* Core.sol - The core access control contract of the ECG , maintains roles and access control.This contract sets initial roles before 
going live. 

* CoreRef.sol - This is a reference to the core contract and defines some modifiers and utilities around interacting with Core.
This has the emergencyAction() function.

* CoreRoles.sol - Holds a complete list of all roles which can be held by contracts inside the Ethereum Credit Guild.

* GuildGovernor.sol - Governor contract based on the OZ governance implementations (inherits and overrides Governor.sol , GovernorSettings.sol
GovernorTimelockControl.sol , GovernorVotes.sol , GovernorCountingSimple.sol) and for access control inferits the Core contracts.

* GuildTimelockController - This contract overrides the OZ Timelock implementation and the access control is handled via the Core contracts.

* LendingTermOffboarding.sol - Think of a lending term as a term which offers loans to a user(borrowers pay collateral in exchange
for CREDIT tokens like gUSDC). Using GUILD the governance token the users can poll to offboard a lending term.No new loans can be issues
on the offboarded lending term.

* LendingTermOnboarding.sol - Similar to what it sounds like , a contract used to onboard a lending term(users can only queue the 
onboarding proposals). To veto this onboarding proposal CREDIT holders can vote to object this(veto the proposal) . A term that
has been offboarded in the past can also be onboarded again.

* ProfitManager.sol - This contract manages all the profits that are generated in the system , the users voting for a lednding term which
 generated a profit will be rewarded through this contract . In case of a loss there is a surplus buffer which is a buffer to
 "soak-up" the losses.
 
* GuildVetoGovernor.sol - This is the governor for the veto mechanism in the ECG. 
"governor will only be able to execute the action of cancelling an action"  This means through this contract only votes against a 
proposal can be casted and the quorum only accounts for the against votes . When enough against votes (above veto quorum) are cast, the veto vote is considered successful

* ERC20Gauges.sol - This is an ERC20 with a gauge style voting mechanism. Support gauge style votes with weights associated with resource allocation. The word lending term can be interchanged with a gauge , user voting for a lending terms means voting for 
a gauge.

* ERC20MultiVotes.sol - an ERC20 extension which allows delegations to multiple delegatees up to a user's balance on a given block.

* ERC20RebaseDistributor.sol - A user (lenders) can choose to enter rebase (their balance is converted to a number of shares internally, and their balance is sharePrice * nShares.
 Then as distributions happen, the share price increases, so their balance increase)
 The contract is an ERC20 with rebase support.
 
* CreditToken.sol - This is the CREDIT token contract (ex - gUSDC for USDC underlying) , This is the debt token of the Ethereum Credit Guild.
Lenders lend USDC -> they are minted gUSDC(CREDIT).

* GuildToken.sol - This is the governance token of the ECG . GUILDis minted through the SurplusGuildMinter (staked)

* AuctionHouse.sol - The auction mechanism of the collateral of a position is handled here(collateral of borrowers is auctioned to cover their CREDIT debt.)

* LendingTerm.sol - The core lending term mechanism contract  , issues CREDIT as debt and takes collateral from the borrower.
interest rate is non-compounding and the percentage is expressed per period of `YEAR` seconds.

* SimplePSM.sol - Peg stability module contract  which maintains the peg of the CREDIT to the underlying (gUSDC and USDC) , allows mint/redeem
of CREDIT token. The amount of CREDIT to be minted (or value of CREDIT to be more precise) is controlled via the creditMultiplier.

* SurplusGuildMinter.sol - This contract mints GUILD for CREDIT tokens , the CREDIT tokens are provided as the first loss capital 
to the profit manager.

* RateLimitedMinter - This contract is used to mint tokens with a rate limit. All the minting throughout the system is done 
through this smart contract.

* RateLimitV2 - An abstract implementation for the rate limit mechanism.

## 1.2 Access Control Roles

There are multiple access controls for different types of functionalities , these roles range from governance functionalities
to timelock functionalities.

Roles -> GOVERNOR 

GUARDIAN 

CREDIT_MINTER 

RATE_LIMITED_CREDIT_MINTER

GUILD_MINTER 

RATE_LIMITED_GUILD_MINTER 

GAUGE_ADD

GAUGE_REMOVE

GAUGE_PARAMETERS

GAUGE_PNL_NOTIFIER

TIMELOCK_PROPOSER 

GUILD_GOVERNANCE_PARAMETERS

GUILD_SURPLUS_BUFFER_WITHDRAW 

CREDIT_REBASE_PARAMETERS

TIMELOCK_EXECUTOR 

TIMELOCK_CANCELLER.

## Architecture Overview:


### Diagram For Governance:

![Diagram for Governance](https://raw.githubusercontent.com/beber89/images/main/sc-interactions.jpg)


1.) The LendingTerm.sol contains the logic for the borrowing of CREDIT tokens . A borrower would deposit collateral in exchange for
the CREDIT tokens and he might/might not be subject to partial repayments , if there were partial repayments enabled on the loan
and the borrower misses a partial repayment then the collateral of the borrower is subject to be "auctioned off" where a bidder can bid
on the collateral in exchange for debt.

2.) A lender can decide wether to enter rebase or not , if entered rebase the balance of the user is converted to shares and 
if the share price increases the balance of the user increases.

Since the Discord server was full of questions I decided to put all of the questions and answers in one place
(to help the community they were shared via twitter https://twitter.com/SakshamGuruji/status/1737733359567356401), these are as follows ->


1.) Governor should make sure he is  sending enough eth in the emergencyAction functionality (to cover all the call operations).
This can be only done by the governor , therefore it makes it a good candidate for centralisation risk.

2.) The greedy algorithm for freeing votes ->

If a user has 3000 tokens, 3 delegations of 1000 tokens each, and attempts to transfer/burn 1500 tokens, the loop will free delegations in chunks of 1000 so the user ends up freeing 2000 voting power, transfers out 1500 tokens, and remains with 500 undelegated votes and only one of the 3 delegations of 1000 tokens still active

3.) Why is there is a distribution period of 30 days when distributing rewards via ERC20RebaseDistributor?

Then anyone could atomically : a.) flashloan USDC somewhere b.) mint gUSDC, enter rebase c.) repay a loan (and the profit that goes to lenders makes everyone rebase up instantly) d.) repay flashloan This would allow the loan repayer to pocket most of the interest if they flashloan a large amount & represent a majority of the rebasing supply

If the rewards are distributed over 30 days, every block only "drips" a tiny amount of rewards (by adjusting the share price through an interpolation), so the user who repays cannot "sandwich" their interest payment with a large deposit/withdrawal into gUSDC

4.) ProfitManager works with multipleLendingTerms

termSurplusBuffer is specific to individual lending terms, and surplusBuffer is a global buffer for the entire system. In the case of a loss, termSurplusBuffer is depleted first, then surplusBuffer, then the creditMultiplier is adjusted down (loss to lenders)

5.) Global debt ceiling is the maximum amount that can be borrowed across all lending terms. In the early period it is used to set a maximum borrowed amount during the guarded launch. In the future it can be used to rate limit borrowing to help protect the protocol against malicious borrows

6.) There's one SimplePSM per CreditToken , There can be various lending terms per CreditToken with different collateral tokens and different parameters. But you only need one SimplePSM per CreditToken to mantain peg with it's pegged token (USDC <> gUSDC

7.) Most users will enter rebase. Our UI will call mintAndEnterRebase for first deposit. It's mainly for smart contracts that might not support rebase, or arbitrageurs that want to minimize gas cost of credit token transfers.

8.) gauge == lending term gauge type is to support multiple markets (all lending terms of the gUSDC market will have type 1, all lending terms of the gWETH market will have type 2, etc)

9.) When a user enters rebase, their balance is converted to a number of shares internally, and their balance is sharePrice * nShares. Then as distributions happen, the share price increases, so their balance increase

10.) 'Lenders' is users who provide USDC to the PSM to get gUSDC and enter the savings rate (enterRebase) 'Borrowers' is users who provide collateral (sDAI, wstETH, etc) to borrow gUSDC in LendingTerms and redeem these for USDC in the PSM.

11.) when staking credit tokens, one can access GUILD tokens in 2 ways : some are minted on the SurplusGuildMinter & can only be used to do gauge votes, some are earned as rewards & go directly in the user's wallet when borrowers pay interests, the SurplusGuildMinter, because it is voting in the gauges with its GUILD tokens, earns some part of these credit tokens as rewards. These rewards are forwarded to the users who staked, and they also get on top of that an amount of GUILD tokens (that is the amount of credit rewards * the reward ratio)

12.) 1 market is composed of: a.) 1 creditToken (e.g: gUSDC) b.) 1 PSM: lender can deposit USDC and get gUSDC, or borrower can redeeem gUSDC against USDC c.) N term: borrow gUSDC against a collateral (sDAI, wstETH, etc). 1 collateral = 1 term d.) 1 ProfitManager e.) 1 RateLimitedMinter (gUSDC)

13.) https://docs.google.com/spreadsheets/d/1yTSHd7AlCQjK5KrMIjHQuH2Ppq7uQeG-FH-oU1ou0-M/edit#gid=153429491 (A spreadsheet explaining user actions)

14.) The borrower themselves could bid in the auction before it reaches market price, because they get returned both the collateral share of the bidder & the borrower (so they recover their full collateral), this could be a rational action to avoid "arbitrage fees" (nobody external will bid until enough collateral of the borrower goes to them to cover both the debt + the gas fees / slippage to do the liquidation)

when someone bid() in the AuctionHouse, the AH calls back to the LendingTerm.onBid(), that calls back to ProfitManager.notifyPnL, that calls back to GuildToken.notifyGaugeLoss if there is a loss, and that is where the slashing happens (more precisely, that is when tokens are 'marked for slashing' and cannot be transferred anymore, then the slashing has to be realized through GuildToken.applyGaugeLoss)

15.) Feature of "imposing periodic partial repayments" (maxDelayBetweenPartialRepay + minPartialRepayPercent) is going to be important, because if lending terms are with uncorrelated assets like wstETH no problem, but if it's on correlated assets, we'll need periodic payments. E.g. for a term that allows to borrow 1.0 gUSDC per 1.05 DAI collateral, with 5% interest, after 1 year the loan will become insolvent (debt = 1.05 USDC worth for 1.05 DAI collateral), so we'll require quarterly payments of 1.25% for instance, and either the position remains healthy forever, either the loan missed a partial payment & can be called. This kind of setup is also better for lenders, because interest is realized regularly, and the profit distribution (rebase savings rate) happens regularly too

16.) Q - is credit multiplier intended to be applied to interest and opening fee (not just principal)? A - yes, the intent is to keep a constant pegToken value for open loans when the creditMultiplier is updated. E.g. if you owed X gUSDC worth 2000 USDC before, you must now owe Y gUSDC still worth 2000 USDC

17.) The profit manager has the ability to split the profits it receives between the surplus buffer (market wide insurance fund), the GUILD stakers, the gUSDC savers, and an optional fourth recipient intended to permit upgrades, for example forwarding profits to an upgraded profit manager, or sending to a DAO treasury separate from the surplus buffer.

18.)termSurplusBuffer is the total staked tokens (first loss capital) surplusBuffer can be thought as the gains the market has made when borrowers repays their loans.

19.) gaugeProfitIndex - index of number of CREDIT rewards per weight allocated to the gauge

20.) If userGaugeProfitIndex is lower than gaugeProfitIndex, it means that user has unclaimed rewards




## 3. Codebase Assessment:

The total number of lines of code in the audit was ~3800 and the overall complexity of the system is pretty high . There are a lot of moving
parts in the codebase , uses OZ governance heavily and it needs a structures approach to make sense of the whole system . 

* The codebase makes use pause functionality for system critical functions like stake and mint.(the guardian pause feature is to prevent 'new usage' of the protocol but not prevent user withdrawals)

* General Readability - Even though the overall system is complex the code has been written in a way that  anyone with basic programming 
 knowledge can make sense  , the variables and comments are well defined and structured.

* Code Style -  The code adheres to a uniform style and indentation pattern, complemented by in-line comments that provide comprehensive guidance throughout it's mechanisms.

* Gas - Throughout the codebase it can be seen that the code was written in a way to optimise gas , this shows the professionalism of the devs.

* Test-Suite - The test suite is complete with more than 97% coverage and well written tests in foundry.

* Errors and Events - Errors and events have been taken care of for every functinality.(Events/Errors are very well defined)

## 4. Level Of Documentation:

The documentation in the c4 audit repo was enough to make sense of the basic moving parts of the codebase , but for a deeper 
understanding of the codebase a more robust documentation is required , this was seen as most of the wardens were confused about 
someof the moving parts in the system such as auctions and interpolation . More graphs with more explanation for every smallest detail
is very much needed.

## 5. Systemic And Centralisation Risks

### 5.1 Systemic Risks

The codebase was battlehard , it had undergone formal verification plus a well structures test suite and all of that made it hard
to find flaws in the system. Although the code was written with security being the top most priority I was sure that there would be 
flaws/bugs in the system due to the complexity. 
There a certain risks which are connected/associated with the systemic design ->

* GUILD holders can make system critical decisions like off boarding / on boarding a lending term and initially
GUILD would be airdropped to the users  , meaning a set of malicious users can make decisions such as offboarding a normal
lending term etc.

* Any malicious actor can make malicious loans , for example a loan where the maximum deplay for partial repayments is very low
and can instantly auctions off the collateral of the borrower.

### 5.2 Centralisation Risks

The ECG has tackled the "centralisation" problem beautifully , they did this by having different roles assigned for different 
functionalities. These roles include -> 

GOVERNOR , GUARDIAN , CREDIT_MINTER , RATE_LIMITED_CREDIT_MINTER , GUILD_MINTER , RATE_LIMITED_GUILD_MINTER , GAUGE_ADD , GAUGE_REMOVE , GAUGE_PARAMETERS , GAUGE_PNL_NOTIFIER
, TIMELOCK_PROPOSER , GUILD_GOVERNANCE_PARAMETERS , GUILD_SURPLUS_BUFFER_WITHDRAW , CREDIT_REBASE_PARAMETERS, TIMELOCK_EXECUTOR , TIMELOCK_CANCELLER.

Also , a lot of the power resides in the hands of the GUILD holders.

But , though it may seem that a lot of the power has been disbursed or decentralised there are still roles (GOVERNOR for example) with
privileges over system critical functions , because of this it should be made sure (for every role) ->

1.) Multisigs are used for privileged roles

2.) Multisigs should come with a timelock

3.) Each key should reside on a different server in case one of the server is compromised.



### Time spent:
32 hours