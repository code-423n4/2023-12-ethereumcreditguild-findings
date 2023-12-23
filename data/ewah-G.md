## Using require statement on creditEarned 

## Impact
consider using require statement on `creditEarned `  
## Proof of Concept

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L432-L435
```
if (creditEarned != 0) {
            emit ClaimRewards(block.timestamp, user, gauge, creditEarned);
            CreditToken(credit).transfer(user, creditEarned);
        }
```


## Recommended Mitigation Steps

```
require(creditEarned > 0, "");
        emit ClaimRewards(block.timestamp, user, gauge, creditEarned);
        CreditToken(credit).transfer(user, creditEarned)
```