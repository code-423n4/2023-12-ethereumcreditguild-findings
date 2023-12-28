Currently the way newSharePrice is calculated in Erc20RebaseDistributor.sol doesn't matches with the comments written in the contract which are as follows 
 ```
        newSharePrice = oldSharePrice * (rebasingSupply + amount) / rebasingSupply
```
The above is what is written in the comments whereas what is implemented is as follows
```
newTargetSharePrice = (amount *
                START_REBASING_SHARE_PRICE +
                __rebasingSharePrice.targetValue *
                _totalRebasingShares) / _totalRebasingShares;
```
As talked with the sponsers they forgot to update the comments so that is why it is low/informational finding otherwise it would be high/medium finding and i would have dig deeper. So update the comments.