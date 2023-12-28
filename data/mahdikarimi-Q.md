## DoS in decreaseGaugeWeight function 
Users may not be able to decrease gauge weight for a while in deprecated gauges,
when a gauge is deprecated, `decreaseGaugeWeight` function will revert until all loans have been cleared and issuance = 0.
in a scenario that there is 250 weight in a deprecated gauge, debtCeiling is also equal to 250, and issuance is 100, 150 of weights are free and users should be able to withdraw them but it reverts until all loans has been cleared.