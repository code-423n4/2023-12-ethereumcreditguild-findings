Analysis Report :-

**Index of table**
| Sl.no  | Particulars  |
|---|---|
| 1   | Overview  |
| 2  | Architecture view(Diagram)  |
| 3  | Approach taken in evaluating the codebase  |
| 4  | Systematic risks  |
| 5 | Mechanism review  |
| 6  | Recommendation   |
| 7  | Hours spend  |



## 1. Overview
The Ethereum Credit Guild represents a groundbreaking innovation in decentralized finance, introducing a comprehensive automation system encompassing borrowing, liquidations, and even governance. This system facilitates predetermined governance calls, significantly streamlining operational processes and enhancing its overall autonomy.


## 2. Architecture view(Diagram)

![ECG DIA](https://user-images.githubusercontent.com/69415766/292615143-f34ba4ce-71cf-4666-b846-b2991ba1c747.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDMzMTgzNjksIm5iZiI6MTcwMzMxODA2OSwicGF0aCI6Ii82OTQxNTc2Ni8yOTI2MTUxNDMtZjM0YmE0Y2UtNzFjZi00NjY2LWI4NDYtYjI5OTFiYTFjNzQ3LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMjMlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjIzVDA3NTQyOVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWI5Y2NlNjljZTI4OWRjZjJhY2Y1ZDAxM2ZlYTdiYWZmMmE2MTBiMGYyMDdiNDA3MTljZGJkZWU5ZGE2YmRlZWUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.hFLcrCOsId95pJlIARpdAPbuKP_HULsAXh2-ms8XrTw)


## 3. Approach taken in evaluating the codebase 
We approached manual code review by looking into each space in the scoped contract and decided to explain some core functional contracts.It is important to note that manual code reviews can be very time-consuming.

**Core Contracta Are:-**

Goverenance

1.GuildTimeLockController.sol
2.GuildVetoGovernor.sol
3.LendingTermOffboarding.sol
4.LendingTermOnboarding.sol
5.ProfitManager.sol

Loan
1.AuctionHouse.sol
2.LendingTerm.sol
3.SimplePSM.sol
4.SurplusGuildMinter.sol

Tokens
1.CreditTokens.sol
2.Erc20Gauges.sol
3.Erc20RebaseDistributor.sol
4.GuildTokens.sol
5.Erc20MultiVotes.sol

**Governance**

1. GuildTimeLockController.sol

This contract governs the time lock actions within the Ethereum Credit Guild ecosystem.It enforeces waiting periods for actions , ensuring security and consensus and prevents immediate execution of potentially risky decison.

2.GuildVetoGovernor.sol

This contract plays vital role in ECG eco and it is the place where guild token holder / credit token holder can veto the on-going proposals by cast vote once it has passed quorum it will considered as successfullt executed.

**Core Function Are:-**

`CreateVeto()`

This function serves as the interface for guild token holders to veto ongoing proposals. It first fetches the block.timestamp associated with the given timelockId and then calls the governor's propose() function to add the veto to the queue.

`executeVeto()`

This function is used to execute the queued veto proposals . First it calls the `_getVetoCalls()` to fetch the target addresses and values by given `timelockID` and calls the governor `execute()` function to execute the queued veto proposals.

3.LendingTermOffboarding.sol
This contract is used to Off-board the lending term if term sounds risky for guild holders or governance.When the term is off-boarded no new loans can issued.

**Core Functions are:-**

`proposeOffBoard()` 

This function is used to offboard a lending term by initiating a voting process. It incorporates multiple validation checks to ensure the term's eligibility for offboarding. If approved, it creates a new poll with an initial voting power of 1 wei and records the term in the lastPollBlock mapping with the current block number.

`supportOffBoard()`

This function allows users to vote for the offboarding of a lending term . It first validates the poll's active status and existence, then retrieves the voter's weight based on their GuildToken holdings. It safeguards against zero-weight votes and duplicate participation, meticulously records the vote with its corresponding weight, and updates the overall poll tally. If the vote accumulation reaches quorum, the term becomes eligible for offboarding. An OffboardSupport event will be emitt.


`offBoard()`

This function allows to offboarding of a lending term that has the sufficient quorum. It first verifies the term's eligibility, then calls the GuildToken contract to remove it as a gauge, effectively disabling new loans and voting. To safeguard protocol liquidity, it conditionally pauses PSM redemptions if this is the first offboarding in progress and redemptions aren't already paused. Finally, it emits an Offboard event.

`cleanup()`

This function is used in the final stages of an offboarded lending term. It have multi-validation system to  verifies that the term has met offboarding requirements, has no outstanding loans, and hasn't been re-onboarded. It then revokes its `RATE_LIMITED_CREDIT_MINTER` and `GAUGE_PNL_NOTIFIER`privileges, conditionally resumes PSM redemptions if applicable, and marks the term as ineligible for further offboarding. Finally cleanup event will be emit.

4.LendingTermOnboarding.sol

This contract is used to create new lending with corresponding parameter and propose on board and it is the lender interface to make some economic activies through the Ethereum credit guild eco-system.

**Core Functions are**

`createTerm()`

This function allows to creation of new lending terms with specified parameters. It meticulously validates the provided parameters to ensure their validity before initializing the term from the implementation address. Finally, it meticulously records the newly created term within the `` mapping datatype for accurate tracking and management.


`proposeOnBoard()`

This function is used to onboarding of new lending terms. It first verifies the term's validity, ensuring it was created by this contract, hasn't been proposed recently, and isn't already active but allowing re-onboarding of previously offboarded terms. It then fetch the proposal data through the `getOnboardProposeArgs()` function, likely encompassing term details and actions, before submitting it to the Governor contract for community governance. The function returns a proposal ID.

5. ProfitManager.sol

This contract plays vital role in Ethereum credit guild protocol. The main functionalities are :-

1. It aims to manages the profits genereated and distribute it to various stake holders (including lenders, GUILD token holders, and potentially other parties).

2. It helps to maintains the surplus buffer to absorb potential losses,acting as a first line of defense against bad debt.

3. It helps to determine the `creditmultiplier` varaible value by profit/loss and total supply of credit tokens.

4. It allow contributors to claims rewards corresponding to their.

**Core functionality**

`notifyPnL()`

This function is used to determine the profit/loss on the protocol and update the `surplusBuffer` and `creditMultiplier`.  First it caches the `surplusBuffer` ,`termsurplus` and `creditmultiplier` if amount argument is below than zero it considerd as loss and calls `notifyGaugeLoss()` to save the `block.timestamp` of the gauge(term). If `loss` is lesser than `surplusBuffer` is subtracts `loss` from it. If `loss` greater than `surplusBuffer` it makes zero value and update the `newCreditMultiplier` by factor of `creditMultiplier` , `loss` and `creditTotaolSupply`. If `amount` is greater than zero considered as profit and determines how the profit is to be distributed according to the profit sharing config.

`claimGaugeRewards()`

This function is used to claim the contributors' rewards. First, it fetches the user gauge weight, gauge profit index, and user profit index, and then validates them. It calculates the credit earned using the user gauge weight and delta index. If the creditEarned is greater than zero, it transfers the rewards to the contributor.



**Loan**


1.AuctionHouse.sol

This contract acts as a marketplace for bidders ready to acquire the collateral of defaulted borrowers by providing the principal amount of the collateral. This helps to maintain solvency and continue to provide loans to borrowers.

**Core Functions**

`startAuction()`

This function can be called by specific users, such as `GAUGE_PNL_NOTIFIER`, to initiate auctions. It employs a multi-validation system to verify the eligibility of a loanId for auction. Upon successful validation, the loanId is added to the `auctions` mapping, serving as a queue for bidders to place bids. 

`bid()`

The bid function allows a user to submit a bid for an active auction. It first verifies the auction's validity and retrieves bid details. It then closes the auction, calls the `onBid()` fucntion of LendingTerm contract of the bid details, and emits an event.

`forgive()`

This function is used to call when a loan auction ends without any bids, even when no credit is requested in return. It marks the debt as a total loss, effectively forgiving the loan. This is typically used when collateral assets are frozen or otherwise inaccessible within the lending term contract. The function closes the auction, calls `onBid()` function LendingTerm contract , and emits an event to the auction's end.


2.LendingTerm.sol

This contract is the facilitating the borrowing and lending activites on ECG eco system.It acts as a container for individual loan agreements, specifying terms like loan amount, repayment schedule, collateral type, and liquidation triggers.It also interacts with the AuctionHouse contract to resolve bad debt through auctions where bidders acquire the borrower's collateral in exchange for repaying the outstanding debt.

**Core functions are**


`debtCeiling()`

This function is used to calculate the maximum amount of debt that can be issued within a specific LendingTerm contract.If no gauge votes exist, the debt ceiling is zero and for each gauges, debt is limited by hard caps or credit minter buffers. By calculations factor in total borrowed credit, gauge weight tolerances, and relative debt ceilings to determine the maximum allowable debt and finally returns the `debtceiling`.


`_borrow()`

This function serves as the interface for borrowers to initiate borrowing activities. It first retrieves necessary variable values such as `creditMultiplier` and `totalBorrowAmount`, and then validates them. Next, it calculates the debt ceiling by taking into account `totalBorrowedcredit`, `gaugeWeight`, and applicable limits. It proceeds to update the loan state, calls the `mint()` function of the rate limit contract to create the borrowed amount, and transfers collateral from the borrower to the lending term.


`_addCollateral()`

This function enables borrowers to add collateral to their loans in order to prevent potential liquidation through auctions. It features a multi-validation system to ensure the accuracy of provided arguments, and subsequently transfers the collateral from the borrower to the loan term.


`partialRepay()`

This function allows partial repayment of an open loan. It allows borrowers to manage their debt and prevent underwater positions due to accrued interest, and it can also be used for periodic repayments mandated by certain lending terms. This function have multiple checks to ensure the loan is valid, open, and not already fully repaid or liquidated. It calculates principal and interest components of the partial payment, verifying that minimum repayment thresholds are met and that outstanding debt remains above minimum borrow levels. The function then updates loan state variables, pull `CREDIT` tokens from the borrower, forwards interest to the ProfitManager, burns principal, replenishes credit minting buffers, and emits an event to signal the partial repayment.


`_repay()`

This function allows the repayment for open loans. It have multi-validation to loan's validity and active status. Then it fetch the total debt, calculates both principal and accrued interest. It then transfer from full debt from the borrower, transfers any accumulated interest to the ProfitManager for distribution, and burns the principal to reduce outstanding credit token. Then, it replenishes the credit minting buffer to ensure future lending capacity. The function proceeds to formally close the loan by updating timestamps and reducing outstanding issuance. It then  returns the borrower's locked collateral, and finally, it emits a LoanClose event with loan ID, closure type, and final debt amount arguments.


`_call()`

This function is allows to initiate the process of calling a loan, which involves auctioning off the collateral to repay outstanding debt. It first performs a series of checks to ensure the loan is eligible for calling, verifying its existence, status, and adherence to calling conditions. If the loan is eligible, the function updates its state with the call time, call debt, and caller information. It then calls `startSuction()`  of AuctionHouse contract to start an auction of the loan's collateral. Finally, it emits a `LoanCall` event.


`forgive()`

This function is used to close a loan when standard repayment or calling processes are failed. This function is suitable for frozen collaterals also. Only a user with the Core Governor role can initiate this process. The function first verifies the loan's existence and open status. It then marks the loan as closed, reducing outstanding issuance and declare the debt as a total loss. Then `notifyPnl()` function is called of ProfitManager contract. To ensure stability, new borrowing is temporarily halted by setting the hard cap to zero. Finally, a `LoanClose` event will be emit.


`onBid()`

This function is callback function from `bid()` function of Auction House contract.This function is used to loan closures through auctions.It will first validates the caller and the loan's eligibility for closure. It then verifies collateral  to ensure its proper handling within the auction. Next, it calculates the profit/loss generated from the loan, taking into account credit received and principal amount. Once these checks are complete, it marks the loan as closed and handles any credit transfers or burns principle amount. Profit/loss are forwarded to the ProfitManager. Outstanding issuance is adjusted to reflect the closure, and collateral is distributed to either the borrower or successful bidder based on auction outcomes. Finally, a `LoanClose` event is broadcast to signal the successful completion of the auction process.


3.SimplePSM.sol

This contract is used to mint and redeem credit tokens independently of lending terms. Users will receive `Credit tokens`, which are pegged tokens.

**Core Function are**

`mint()`

This function allows the user to mint credit token in excahnge of thier underlying tokens and minted credit tokens are transfered in form rebase to stakers.


`mintAndEnterRebase()`

This function allows users to both mint Credit tokens and immediately start earning the savings rate through a rebasing mechanism. It have check to ensure the user isn't already engaged in a rebase. Next, it precisely calculates the amount of Credit tokens to be minted based on the provided input of underlying tokens. The user's underlying tokens are then transferred to this contract. Then it calculates amount of Credit tokens and credits them directly to the user's account. Then is calls `forceEnterRebase()` function to forcefully activates the rebase mechanism for the user, enabling them to start earning interest on their newly minted tokens without delay. Then it finally emits `Mint` event.


`redeem()`

This function is used to redeem their credit tokens back to underlying tokens.Then it calculates the exact amount of underlying tokens the user will receive in return for the specified number of Credit tokens being redeemed.Then user's credit tokens will be burned and reduces the overall supply of these tokens in circulation.Then underlying amount will be calculated and transferred to the user.Finally `Redeem` event will be emit.



4.SurplusGuildMinter.sol

This contract empowers users to mint GUILD tokens directly from their Credit tokens, unlocking a pathway to participate in the governance of the Ethereum Credit Guild. By acquiring GUILD governance tokens, users gain the right to cast votes on crucial proposals, shaping the direction and evolution of the ecosystem. This mechanism fosters a decentralized decision-making process, ensuring that the community has a collective voice in shaping the future of the guild.


**Core functions are**

`stake()`

This function enables users to stake their Credit tokens in exchange for GUILD tokens. It first validates the provided arguments, then transfers Credit tokens from the user to the contract. It subsequently calls the `donateToTermSurplusBuffer()` function to transfer the amount to the ProfitManager contract. It then calculates the appropriate `guildAmount` and calls the `mint()` function of the RateLimiter contract to mint `GUILD` tokens. Finally, it updates the user's state to reflect the transaction.


`unstake()`

This function serves as the inverse of the stake function, enabling users to redeem their Credit tokens in exchange for `GUILD` tokens. It first retrieves the user's stake status by calling the `getRewards()` function and verifies whether the user has been slashed. It then calculates the user's mint ratio and the corresponding GUILD amount to decrement the gauge and replenish the buffer. Finally, it withdraws the specified amount of Credit tokens and transfers them to the user's account.This function is act as opposite to stake function allows to redeem the credit tokens by guild tokens.First it fetch the user stake status by calling the `getRewards()` function and check whether the user slashed or not and calculates the user mint ratio and guild amount to decrement and replenish the buffer and finally withdraw amount of credit tokens and transfer it to user.

**Token**

1.Erc20Gauges.sol

This abtract contract is holds the mechanism of gauge style voting with liquid weights.Gauges represent potential resoureces (token emssion and credit line access).Holders of the GUILD token can allocate their voting weight to different gauges to influence resource distribution.

**Core functions are**

`calculateGaugeAllocation()`

This function is used to determining the fair share of a resource that should be directed to a specific gauge. It verifies whether the given gauge is active or not. If it's found to be inactive, the function gracefully returns a zero allocation. Next, it fetchs the total voting weight. If this total weight is found to be zero, again returns zero, ensuring no allocation occurs in such a scenario. Undeterred, it then proceeds to extract the specific weight assigned to the individual gauge, carefully considering its relative influence within its type. With these crucial values in hand, it elegantly executes a precise calculation to determine the gauge's rightful allocation, employing the equation` (quantity * weight) / total`. Finally, the function gracefully returns the calculated allocation, ensuring a fair and proportional distribution of the resource based on the gauge's voting power.


`_decrementWeightUntilFree()`

This function is used to  weight availability for token operations. It frees up weight from gauges, prioritizing efficiency over pinpoint precision through a greedy approach. First, it checks if the user already possesses sufficient free weight, gracefully exiting if so to avoid needless work. If weight needs to be free, it prepares tracking variables to accurately monitor the weight liberated for both the individual user and the overall system. Then, it embarks on a methodical journey through each of the user's gauges, both active and inactive ones, assessing the weight allocated to each. Upon a gauge with weight assigned, it frees the entire weight, effectively divesting the user's influence on its voting power. If the gauge remains active, it rectifies the total weight of its type, ensuring accurate collective voting power. With each liberated weight, it meticulously updates the accumulated freed weight for both the user and the overall system. Once its careful sweep through the gauges concludes, it gracefully adjusts the total weight to reflect the weight reductions, thus ensuring ample space for the impending token operations.

2. Erc20MultiVotes.sol

This abstract contract is used to holds the mechanism of multi voting system through delegation to multiple delegatees up to a user's balance on a given block.


3. Erc20RebaseDistributor.sol

This abstract contract plays vital role in ECG eco system by holding the mechanism of rebasing. This contract is used  to automate the distribution of rebase rewards to holders of a specific ERC20 token. All the activites in the ECG ego transfer and transferFrom is based to rebasing the tokens.



## 4. Systematic risks

The Ethereum Credit Guild lacks an upgrade mechanism for addressing financial and security risks within its contracts. While it does have a mechanism to update core roles, this doesn't extend to contracts themselves. This poses a significant concern, as the Profit Manager contract plays a crucial role in the ECG ecosystem. If this contract were to contain manipulated code, it would introduce vulnerabilities to the Lending Term Contract, Surplus Guild Minter Contract, and other contracts that lack setter functions for the Profit Manager. These contracts would remain exposed to the vulnerable code until the Governor became aware of the issue.



## 5.  Mechanism review

After manual testing, we concluded that the Ethereum Credit Guild Protocol has an innovative rebasing mechanism that allows lenders to engage in economic activities by providing collateral to borrowers under specific terms and conditions. It heavily relies on an optimistic or veto-based governance model. Majority/Minoirty holders of Guild tokens (governance tokens) can make decisions regarding lending term proposals, such as onboarding or offboarding a term if it's deemed risky. They also determine the interest rate split between GUILD stakers, lenders, and the surplus buffer. Users who contribute to the protocol by participating in the rebasing process through staking credit tokens immediately start earning savings rates and can claim rewards based on specified rates. Guild token holders who vote for lending terms that result in bad debt face penalties in the form of token slashing and loss of rewards.The protocol is still under development and relies on complex smart contracts, which could be vulnerable to technical glitches or security exploits. The success of the protocol depends heavily on the decisions made by Guild token holders. Poor governance decisions could lead to negative consequences for the protocol and its users.



Comparison Table ECG and MarkerDAO

|Feature   | Ethereum Credit Guild (ECG)| MakerDAO (MKR)|
|---|---|---|
| Focus  |Credit scoring and lending in DeFi   | Stablecoin issuance and governance  |
| Mechanism  | Rebase on token based on credit market performance  | Algorithmic stablecoin (DAI) backed by collateralized debt positions (CDPs)  |
| Goals  | Improve access to credit in DeFi, generate yield for token holders  | Maintain DAI price peg to USD, enable decentralized governance of Maker Protocol  |
| Centralised Risks  | Potential for centralized governance, and upgrades roles  | Decentralized oracles, on-chain governance through MKR voting, limited upgradeability  |
| Token Use Cases  | Governance, staking for rewards, access to credit products  | Governance, incentivizes stablecoin issuance and stability, collateral for CDPs  |
| Risks  | Rebase mechanism volatility, dependence on credit market health, potential governance manipulation  | Algorithmic risks, potential for black swan events impacting DAI peg, smart contract vulnerabilities  |
| Target Audience  | DeFi borrowers and lenders, credit scoring participants  | Stablecoin users, DeFi enthusiasts, cryptocurrency investors  |
| Strenghts  | Innovative approach to credit scoring in DeFi, potential for high yield generation  | Proven track record of DAI stability, strong community governance, secure smart contracts  |
| Weakness  | Relatively new project with higher uncertainty, complex rebase mechanism can be difficult to understand  | Algorithmic stablecoins inherently carry risks, potential for governance gridlock  |




## 6. Recommendation
After manual observation of code and mechanism we recommend a simple changes which make more robustness to Ethereum Credit Guild protocol.

1. According to our analysis, a single ProfitManager contract manages the profits generated and distributed within the system among various stakeholders with different lending terms. The ProfitManager contract features a credit multiplier, which allows for leverage (2x, 3x, etc.) to borrowers based on the set value. The creditMultiPlier is used to determine the principal amount during `borrow()`, `partialRepay()`, `repay()`, and `forgive()` functions.

The value of `creditMultiplier` is determined by the profit/loss and total supply of credit tokens. If a loss occurs that exceeds the surplusBuffer, the `borrow()`, `partialRepay()`, `repay()`, and `forgive()` functions will be halted. This poses a significant risk for the Ethereum Credit Guild, as it can prevent borrowing when the protocol is already in a loss state. It also blocks users from partial or full repayments, and even governance cannot forgive loans because the `creditMultiplier` has a zero value due to the loss. Solidity reverts when a division is attempted with a zero denominator.

Our recommendation is to add a minimum value to the creditMultiplier when the protocol is in a loss state, even if it's zero. This would enable borrowers to partially or fully repay their debts, rather than resorting to auctions.

Recommedation :-
```solidity
function forgive(bytes32 loanId) external onlyCoreRole(CoreRoles.GOVERNOR) {
	...
	// mark loan as a total loss
        uint256 creditMultiplier = ProfitManager(refs.profitManager)
            .creditMultiplier();
            if(creditMultiplier == 0) creditMultiplier = MINIMUM_CREDIT_MULTIPLIER;//@audit check here
        uint256 borrowAmount = loans[loanId].borrowAmount;
        uint256 principal = (borrowAmount *
            loans[loanId].borrowCreditMultiplier) / creditMultiplier;
            
            ...
}


function _repay(address repayer, bytes32 loanId) internal {
	// compute interest owed
        uint256 loanDebt = getLoanDebt(loanId);
        uint256 borrowAmount = loan.borrowAmount;
        uint256 creditMultiplier = ProfitManager(refs.profitManager)
            .creditMultiplier();
            if(creditMultiplier == 0) creditMultiplier = MINIMUM_CREDIT_MULTIPLIER;//@audit check here
        uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /
            creditMultiplier;


}

    function _partialRepay(
        address repayer,
        bytes32 loanId,
        uint256 debtToRepay
    ) internal {
    
         uint256 creditMultiplier = ProfitManager(refs.profitManager).creditMultiplier();
         if(creditMultiplier == 0) creditMultiplier = MINIMUM_CREDIT_MULTIPLIER;//@audit check here
        uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /
            creditMultiplier;
    
    }

```

2. As we know, a single ProfitManager contract manages the profits generated and distributed within the system among various stakeholders with different lending terms. If any security risk or financial error occurs, apart from the lending terms contract, other contracts that depend on the profit manager will also be at risk. Our recommendation is to implement an upgradeable mechanism for crucial contracts. We suggest adopting the UUPS (Universal Upgradeable Proxy Standard) for crucial contracts, allowing for upgrades even if various risks occur.Consider the contract's upgradeability to address potential vulnerabilities or add new features. Add setter function to the contract which dont't have incase of profit manager have vulnerable code then it can set to new profit manager without affecting the implemented contract (like surplus guild minter and lending term).



3. Add a mechanism to Lending term if partial repayment is allowed then has to pay with minimum amount . If partial repayment is not required then borrwer has to be repayment full .In below POC code snippet you can see  `minPartialRepayPercent` set as zero means partial repayment has no size it would allows borrowers to pay as little as they want and considered as partial repayment and function `partialRepayDelayPassed()` doesn't give true because borrwer already paying partial amount and loan cannot be called for auction also.

Recommendation:-

Implement mechaism before creating the term whether the partial repayment to be allowed or not by adding boolean data type. If partial repayement is true then `minPartialRepayPercent` has to be minimum value while intialising the term.

POC link :-
https://github.com/pavankv241/BlockChain-in-Banking-Sector/blob/main/POCECG.sol

Paste the above POC in LendingTerm.t.sol file.




## 7. Hours spend

90 Hours 

(Qa + Analysis + medium issue)

Note :- Considered the following points to judge

Highlights of the report 

1.Representing the smart contract working flow through the diagram. 

2.Extensive analysis of contract by contract and function by function analysis. 

3.Comparison table with other competitive protocol 

4.Best recommendation which could safely implemented.



### Time spent:
87 hours