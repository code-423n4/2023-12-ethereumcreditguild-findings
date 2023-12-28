[I-01]: Wrong comment

According to the comments in `ERC20Gauges.sol` ECG made the following change:
`- Consistency: make incrementGauges return a uint112 instead of uint256` [L49](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L49)
However, the return value of `incrementGauges` has remained `uint256`:
```
    function incrementGauges(address[] calldata gaugeList, uint256[] calldata weights)
        public
        virtual
        returns (uint256 newUserWeight)
        {
```
[L269-L272](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L269-L272)

Recommendation: remove the comment 
```
- Consistency: make incrementGauges return a uint112 instead of uint256
```
[L49](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L49)