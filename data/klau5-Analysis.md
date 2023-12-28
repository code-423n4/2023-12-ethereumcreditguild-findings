# Approach taken in evaluating the codebase

| Stage | Detail |
| --- | --- |
| Compile and run test | Clone the repository, install dependencies, set environment and run test. |
| Check scope and sort | List the scopes, sorting them in ascending order. Identify core and peripheral logic. |
| Read docs | Read code4rena README and docs. Research on who’s sponsors and what’s their goal. Review the sponsor's existing projects, if any. |
| Understand core logic | Skim through the code to understand the core logic. |
| Manuel Code Review | Read the code line by line to understand the logic. Find bugs due to mistakes. |
| Organize logic with writing/drawing | Organize business logic and look for logic bugs. |
| Write report, PoC | Write the report and make PoC by using test code. |

# Architecture recommendations

If a bad dept occurs, all weights for that gauge are removed. Even a small loss will result in the deletion of all investments. This seems too harsh to investors. We suggest applying mitigation when a termSurplusBuffer is left, such as some kind of defense, or applying a proportional loss.

# Centralization risks

In this protocol, most of the power is granted to the DAO. Since actions are determined by voting, it can be considered quite decentralized.

However, the quorum is set to a fixed value and does not automatically increase as totalSupply increases. Therefore, if the quorum is set to less than 50% of totalSupply, the likelihood of a 51% attack is higher than normal. Also, since veto is possible with a relatively small quorum, there is a possibility that the DAO can be controlled by a small number of people. So it is important to be careful with quorum control.

# Mechanism review

## Credit token

A credit token is a token that is pegged to an external token. The rebase feature can be turned off or on individually. Users who turn off the rebase feature can use it as normal ERC20, while users who turn it on will automatically rebase when they move their tokens and receive additional credit tokens based on the changed share price.

When you receive a loan from LendingTerm, it mints credit tokens. When you repay the loan, you must pay the amount including interest in credit tokens. The auction for the bad dept also uses credit tokens. You can pay bids with credit tokens and receive collateral tokens.

The credit token also has a voting function. You can use it to cast a veto vote on DAO or LendingTermOnboarding votes.

### ERC20MultiVotes

It is inherited by the Credit Token and is responsible for the voting function. As long as you own an ERC20 token, you can vote. In order to vote, you must delegate to yourself or another user. You can split and delegate to multiple delegatees. Vote counting is based on checkpoints. A checkpoint is created each time the delegation is updated.

If you lose tokens by transferring or burning, and if you don't have enough free votes, it release your delegated votes to get free votes before proceeding. There is no guarantee who will be released.

### ERC20RebaseDistributor

It is inherited by the Credit Token and is responsible for the rebase function. Rebasing can be turned off or on, and users with rebasing turned on will automatically rebase their tokens whenever they move.

The ProfitManager calls the `distribute` function whenever there is a profit from paying interest for loan. Calling the `distribute` function will increase the price of the share and mint the credit token upon rebasing.

### SimplePSM

When you deposit the pegged token, credit tokens are minted. Initially, the exchange is 1:1, but if there is a loss in the lending term, the multiplier of the credit token drops and more credit tokens are minted. You can burn the credit token and get the pegged token back.

### RateLimitedMinter

When you make a loan, the loan is delivered in the form of minting credit tokens. When the loan is repaid, credit tokens equal to the principal amount are burned. The interest is sent to the profit manager to be used as a reward or as a buffer.

To mint the loans, it use a RateLimitedMinter. RateLimitedMinter limits the number of tokens that can be minted per second to prevent suddenly minting large amounts of tokens.

## Guild token

The Guild token is the governance token used by the entire system. It can be used to vote for GuildGovernor, LendingTermOnboarding, and LendingTermOffboarding, as well as to vote for GuildGovernor veto and LendingTermOnboarding veto.

Guild token holders can invest by assigning a weight to the gauge of each lending term based on the amount of tokens they own. If the loan makes a profit by paying interest, they will receive a reward equal to their weight allocation. However, if the loan loses money, you will lose the amount of guild tokens you have weighted.

As with the Credit token, it inherits from ERC20MultiVotes to implement the voting functionality.

### ERC20Gauges

It is inherited by Guild tokens and implements the ability to invest in each lending term. You can distribute weight to each lending term as much as the amount of tokens you own. If the lending term is profitable, you can receive credit tokens as a reward in proportion to your weight.

If a lending term loses, you lose as many guild tokens as the weight you allocated to this lending term and the weight is reset.

If you lose tokens by transferring or burning, if you don't have enough free weight, unallocate to get more free weight before proceeding. There is no guarantee who will be released.

### SurplusGuildMinter

You can invest in a specific lending term by staking credit tokens. When interest is accrued, guild tokens are minted in addition to the credit tokens generated by the interest. To mint guild tokens, it uses the RateLimitedMinter. If there is a loss in the lending term, all stakers will lose their staked credit tokens and be forced to unstake.

## Governance

It is mainly composed of GuildGovernor (DAO) and LendingTermOnboarding. GuildGovernor is the central governance where guild token holders can freely propose and vote to run the entire system. LendingTermOnboarding is a vote to list a new lending term. Voting for GuildGovernor and LendingTermOnboarding can be done by guild token holders. If a vote is successful and the action is pending, a veto vote can be cast. veto votes can be cast by guild token and credit token holders respectively. A successful veto vote can cancel a pending proposal.

In addition, there is a LendingTermOffboarding governance. This can only be voted on by guild token holders and is responsible for taking down previously listed lending terms.

## Lending

### Lending term

It makes loans by minting credit tokens in exchange for collateral tokens. When the interest is repaid, it is sent to the profit manager and distributed to guild holders who have been assigned gauge weight, credit tokens for rebasing, and stored in a buffer.

The interest rate on a loan is based on a one-year term. The opening fee is the interest charged immediately after the loan is made.

The bad dept can call and send it to auction. The auctioned loan receives credit tokens as bids and sells the collateral. The bidders are given the collateral and the remaining collateral is given to the lender.

If the bid is lower than the principal amount of the loan, the protocol has lost money, and those who invested in that lending term gauge will have their weight in guild tokens burned, and SurplusGuildMinters will be slashed.

### AuctionHouse

A contract to auction off bad dept. Up until the midpoint, bids are fixed (equal to the remaining debt) and the number of collateral tokens that can be get increases. After the midpoint, all collateral can be taken, and the required bid decreases.

### Time spent:
54 hours