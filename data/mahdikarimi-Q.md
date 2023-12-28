## DoS in decreaseGaugeWeight function 
Users may not be able to decrease gauge weight for a while in deprecated gauges,
when a gauge is deprecated, `decreaseGaugeWeight` function will revert until all loans have been cleared and issuance = 0.
in a scenario that there is 250 weight in a deprecated gauge, debtCeiling is also equal to 250, and issuance is 100, 150 of weights are free and users should be able to withdraw them but it reverts until all loans has been cleared.
following line reverts: 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L227-L230