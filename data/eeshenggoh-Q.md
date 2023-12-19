The way the `ERC20RebaseDistributor::transfer`, `ERC20RebaseDistributor::transferFrom` & `ERC20RebaseDistributor::_burn` function is coded can be optimized by ensuring the execution of code be minimized. Instead of calling the _mint function, can just combined to one transfer function with deducted amount if required.
```diff 
File: src\tokens\ERC20RebaseDistributor.sol::transfer
            uint256 mintAmount = rebasedBalance - fromBalanceBefore;
            if (mintAmount != 0) {
--                ERC20._mint(msg.sender, mintAmount);
                fromBalanceBefore += mintAmount;
                decreaseUnmintedRebaseRewards(mintAmount);
                emit RebaseReward(msg.sender, block.timestamp, mintAmount);
            }
        }

        // do ERC20.transfer()
--      bool success = ERC20.transfer(to, amount);
++      bool success = ERC20.transfer(to, amount - mintAmount);
```
```diff
File: src\tokens\ERC20RebaseDistributor.sol::transferFrom
            uint256 mintAmount = rebasedBalance - fromBalanceBefore;
            if (mintAmount != 0) {
--              ERC20._mint(from, mintAmount);
                fromBalanceBefore += mintAmount;
                decreaseUnmintedRebaseRewards(mintAmount);
                emit RebaseReward(from, block.timestamp, mintAmount);
            }
        }

        // do ERC20.transferFrom()
--      bool success = ERC20.transferFrom(from, to, amount);
++      bool success = ERC20.transferFrom(from, to, amount-mintAmount);

```
```diff
File: src\tokens\ERC20RebaseDistributor.sol::_burn
            uint256 mintAmount = rebasedBalance - balanceBefore;
            if (mintAmount != 0) {
                ERC20._mint(account, mintAmount);
                balanceBefore += mintAmount;
                decreaseUnmintedRebaseRewards(mintAmount);
                emit RebaseReward(account, block.timestamp, mintAmount);
            }
        }

        // do ERC20._burn()
--        ERC20._burn(account, amount);
++        ERC20._burn(account, amount-mintAmount);

```