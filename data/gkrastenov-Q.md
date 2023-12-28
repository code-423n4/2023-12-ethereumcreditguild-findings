# [L-01] Wrong constanst for POLL_DURATION_BLOCKS
The wrong value is set for the constant variable `POLL_DURATION_BLOCKS`. Currently, in the Ethereum blockchain, a new block is created every 12 seconds. 46523 blocks do not represent 7 days; they are around 6 and a half days, which is less time than the expected 7 days. Additionally, this constant can be invalid if the contract is deployed on other blockchains leading to very big or small vlaue for maximum time for proposla to be approved.

# [L-02] Missing check if loans is called
Before forgiving the loan, check if the loan has already been called. If the governor tries to forgive loans because regular repayment is blocked, everyone can front-run their transaction and call the loans.

# [I-01] Typos in the codebase
In many places in the codebase, small typos are made. Review the comments again and fix them. Most of the typos are unclosed brackets, for example.

```diff
-params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%
+params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100%]

- // 1st call: guild.addGauge(term)
+ // 1st call: guild.addGauge(type, term)
targets[0] = guildToken;
```