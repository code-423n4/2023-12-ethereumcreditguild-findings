# Ethereum Credit Guild Analysis

## Index
- [Ethereum Credit Guild Analysis](#ethereum-credit-guild-analysis)
  - [Index](#index)
  - [1. Approach](#1-approach)
  - [2. Mechanism Flows](#2-mechanism-flows)
  - [3. ECG in the wider defi lending space](#3-ecg-in-the-wider-defi-lending-space)
  - [4. Risks](#4-risks)
  - [5. Codebase Quality](#5-codebase-quality)
  - [6. Areas of Concern ](#6-areas-of-concern)
  - [7. Conclusion](#7-conclusion)


## 1. Approach

I formulated multiple stages with clearly defined goals in order to have a clear purpose and structure for the audit.

|Stage | Activity| Time Spent |
|---|---|---|
|Preparation| When the contest was announced, I dedicated one week to better understand the defi lending space. I spent 3 days on Aave/Compound and MakerDao. This gave me a basic insight on how defi lending works and immensely helped me in my first lending audit.    |21h|
|Reading| I started with documentation. After which, I went over the code 3 times bottom to top while taking as many notes as possible. The two main complexity for me were the governance structure and the calculations of the rebase mechanism|43h|
|Drawing|In order to clarify the vague mental model of the protocol that I currently had after the reading stage, I created diagrams that illustrated some of the main mechanisms of the protocol|9h|
|Investigation|By now I had a very long list of questions and preliminary findings so I took each item and investigated until I was certain I had the answer and/or I had clearly identified an issue. This involved heavy use of Foundry tests.|32h|
|Reports| One by one I wrote the submissions for all my findings and prepared foundry poc's where I deemed it necessary|18h|
|Total||119h|

## 2. Mechanism Flows

I created a number of diagrams to better illustrate how certain important mechanism function in the ECG system. 

### How does Liquidity get provided to the system? 
[![image](https://github.com/14si2o/C4-ECG/assets/111807030/da272cb4-9178-413b-9640-704611f9c659)](https://asset.cloudinary.com/dkwl8aepc/825c90185de894db2f801c3c81bacf88)

### What is the rebasing flow?
https://asset.cloudinary.com/dkwl8aepc/22b0d833ff69d4f0bc593259b05b963e

### How does an auction work?
https://asset.cloudinary.com/dkwl8aepc/555c62c7ecb2ce64f9f2c4cae60edfb6


### How do lenders get profit?
https://asset.cloudinary.com/dkwl8aepc/06caf4b4315578e65aeda6f7102abab0

### What is the offboarding flow?
https://asset.cloudinary.com/dkwl8aepc/1a66929965813130b79f833781c6b312


## 3. ECG in the wider defi lending space

A critical question any defi lending question tries to answer is: 
 - How do we determine the correct value of collateral? 

The correct value of collateral determines whether a loan is a happy source of profit or a disastrous bomb of bad debt just waiting to explode. It makes or breaks a lending protocol. 

The first large lending protocols such as Aave, Compound and MakerDao all chose to delegate this task to an Oracle, a trusted external source of information. 

Oracle protocols such as Chainlink often aggregate price information from many different sources to minimize the risk of price manipulations or stale prices. 

Yet millions and millions are lost every year to due to exploits related to oracle manipulation.

The Ethereum Credit Guild is part of a new wave of lending protocols that try to answer the question of value without relying on external pricefeeds. 
 - Ajna relies on permissionless pool creation, price specified lending and market-derived interest rates. 
 - Timeswap, deployed on polygon, relies on a 3-variable AMM (principal pool X, interest rate pool Y and collateral factor pool Z) with arbitrageurs maintaining correct prices. 
 - ECG uses the novel approach of markets linked to one pegtoken and n number of lending terms, with governance actors having an economic incentive to promote good terms and offboard bad ones. 


## 4. Risks
### Centralisation Risks

 1. **GUILD Holders**:  Upon deployment, a number of GUILD tokens will be distributed to investors, protocol members and other actors. Depending on the allocation size, there exist a risk that a disgruntled GUILD holder might be able to veto governance proposals without risking a loss since he has not made an investment in cTokens.
 2. **GUARDIAN**: The guardian role has extreme powers to pause the protocol and cancel any proposal. If the multi-sig of the protocol team would be compromised, the protocol could be easily held hostage. 
 
### Economic Risks
 1. **Liquidity and Debt**: The liquidity present in the PSM module is not 
used in any calculation of the debt ceiling of the market. This is extremely dangerous since this makes it possible for borrowers to take out loans that cannot be redeemed. 

### Governance Risks
 1. **Disjointed Procedures**: Multiple governance actions consist of creation, voting and execution of a proposal. However, often there is no automatic execution of the next step in the procedure, even when this would open up opportunities for frontrunning attacks. 
 2. The quorums to approve a proposal or a veto are hardcoded at deployment, but the GUILD distribution at day 0 is unknown. It is therefore very difficult to say if these quorums are set correctly or if there is an attack vector possible. 

## 5. Codebase Quality
### Documentation

The documentation provides a good high-level overview of the system with an extremely detailed architecture diagram. This could be expanded with a more detailed description of the individual contracts.  

### Testing

There is a very high test coverage, which is very good. However, the tests only describe the most straightforward scenarios. There are no tests with multiple actors lending and borrowing, simulating an actual market environment. 

### Clean Code

Almost all functions have NatSpec and there are small comments throughout the code illustration the purpose of small pieces of code. So the code is quite clean. 

## 6. Area's of Concern
The protocol suggest a number of attack ideas. In this section I answers to those questions for which I believe to have done sufficient investigation. 

### Attack Ideas
 - **Malicious Collateral**: there is a legitimate attack vector with an ERC20 collateral token with massive impact, which I have submitted as a medium finding. However, the likelihood of this happening is extremely low. 
 - **Messing up creditMultiplier**: I have not found any realistic way to manipulate the creditMultiplier to the advantage of an attacker. If the creditMultiplier gets reduced to a minute amount, there is the risk it rounding down to 0 and bricking the contract. But this would require +500.000 losses of 10% of the market, which is ludicrous.
 - **Auction Gaming**: The exact idea mentioned in the readme is not findable, but I have found a way in through a lender can buy his collateral back at a reduced loss through the auction system. This finding was submitted as a high.   

## 7. Conclusion

The Ethereum Credit Guild provides a novel approach to the idea of lending in defi combining open governance, with almost full free reign for actors, optimistic vetocray and economic incentives for them to be good shepherds. 

An extremely complex puzzle which is very difficult and tricky to get right. There are some challenges as I have outlined in the risk part, but I do not believe these are impossible to overcome. 

It has been absolute pleasure working with Eswak and OneTrueKirk, very rarely have I audited with with a protocol team that was this helpful and responsive!
 

### Time spent:
119 hours