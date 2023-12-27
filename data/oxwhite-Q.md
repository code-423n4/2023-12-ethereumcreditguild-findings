## Low Issues
#### L1: Missing Docstrings
Throughout the codebase there are some structs  that do not have docstrings. For instance:
Line77-SurplusGuildMinter.sol 
Line43-ERC20MultiVotes.sol
Line40-AuctionHouse.sol
Consider thoroughly documenting all structs and their parameters  even if those are straightforward. It would be better in terms of codebase overall clarity and readability.
#### L2:Inconsistent naming convention.
In contract ProfitManager.sol,  when updating profitSharingConfig in the function setProfitSharingConfig(), "creditSplit" is used as a parameter but not used in profitsharingConfig.
related line:   https://github.com/volt-protocol/ethereum-credit-guild/blob/main/src/governance/ProfitManager.sol#L197
Recommendation:profitSharingConfig = ProfitSharingConfig({
    surplusBufferSplit: uint32(surplusBufferSplit / 1e9),
    creditSplit: uint32(creditSplit / 1e9),  // <-- Use the same name
    guildSplit: uint32(guildSplit / 1e9),
    otherSplit: uint32(otherSplit / 1e9),
    otherRecipient: otherRecipient
});`` and also restructure the related struct(profitSharingConfig) or removing creditSplit from the params.



## Notes-Additional Informations

#### N-01:Unused Variable
In SurplusGuildMinter.sol the variable YEAR is unused and could be removed.
Consider removing unused variable to improve the overall clarity and readability of the
codebase
#### N-02:Misleading Events' Name
In ProfitManager.sol , event MinBorrowUpdate()  is causing confusion by its name. The event is used inside the constructor simply logging out "minBorrow" info not updating anything . It would be better to use a different event name  in the constructor and use event "MinBorrowUpdate"  in the function called SetMinBorrow() with logging off both oldValue and newValue.
ın ERC20Gauges.sol, there is ambiquity with the names of events "IncrementGaugeWeight and DecrementGaugeWeight". At first sight, they imply that user's gauge weight is increased/decreased in a specific gauge , however in the related functions they are used, ıt is obvious both gauge and user weight are affected. It would be better to name them "IncrementGaugeAndUserWeight, DecrementGaugeAndUserWeight"