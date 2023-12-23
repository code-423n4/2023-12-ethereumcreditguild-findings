## [ L -01]  Updating If Else Statement. 

## Impact

Consider updating < in if  statement with <= to allow filling the cap fully, while cutting off else statement as it wont be necessary. Also updating emit. 

##  Codes

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L315-L328

```
if (loss < _surplusBuffer) {
                // deplete the surplus buffer
                surplusBuffer = _surplusBuffer - loss;
                emit SurplusBufferUpdate(
                    block.timestamp,
                    _surplusBuffer - loss
                );
                CreditToken(_credit).burn(loss);
            } else {
                // empty the surplus buffer
                loss -= _surplusBuffer;
                surplusBuffer = 0;
                CreditToken(_credit).burn(_surplusBuffer);
                emit SurplusBufferUpdate(block.timestamp, 0);
```



## Tools Used

Manual review

## Recommended Mitigation Steps
``if (loss <= _surplusBuffer)
``
``emit SurplusBufferUpdate(block.timestamp, _surplusBuffer);
``

