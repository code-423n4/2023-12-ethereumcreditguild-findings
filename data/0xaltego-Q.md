|        | Issues                                                                                                                                      |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| [L-01] | Panic revert possibility |
| [L-02] | Inability to delegate and permit                                                      |
| [L-03] | No term validation                                                                                     |
| [L-04] | Functions expected to be called                                                                       |
| [L-05] | Bribing contract / whale domination                               |
| [N-01] | Incompatibility with EIP4337                                                          |
| [N-02] | The protocol is not compatible with high decimal tokens                                                                             |
| [N-03] | Blocktime consistency                           |



## [L-01] Panic revert possibility

`updateTotalRebasingShares` doesn not validate `currentRebasingSharePrice` being less than `val.targetValue` which might cause panic revert.
```solidity
Contract: ERC20RebaseDistributor.sol

210:         InterpolatedValue memory val = __rebasingSharePrice;
211: -->        uint256 delta = uint256(val.targetValue) - currentRebasingSharePrice;//@audit-info
212:         if (delta != 0) {
```
## [L-02] Inability to delegate and permit 
If the Guild Tokens is transferrable and planned to be used in voting, it will not be possible to use the `permit` and `delegateBySig` together due to both use `_useNonce`, and the nonce will change once one of them is called.

## [L-03] No term validation
There's no rterm validation in stake() which might cause users not to profit anything due to the staking in non-existing terms.

```solidity
Contract: SurplusGuildMinter.sol

114:     function stake(address term, uint256 amount) external whenNotPaused {
115:         // apply pending rewards
116:         (uint256 lastGaugeLoss, UserStake memory userStake, ) = getRewards(
117:             msg.sender,
118:             term
119:         );
120: 
121:         require(
122:             lastGaugeLoss != block.timestamp,
123:             "SurplusGuildMinter: loss in block"
124:         );
125:         require(amount >= MIN_STAKE, "SurplusGuildMinter: min stake");
126: 
127:         // pull CREDIT from user & transfer it to surplus buffer
128:         CreditToken(credit).transferFrom(msg.sender, address(this), amount);
129:         CreditToken(credit).approve(address(profitManager), amount);
130:  -->    ProfitManager(profitManager).donateToTermSurplusBuffer(term, amount);
131: 
132:         // self-mint GUILD tokens
133:         uint256 _mintRatio = mintRatio;
134:         uint256 guildAmount = (_mintRatio * amount) / 1e18;
135:         RateLimitedMinter(rlgm).mint(address(this), guildAmount);
136:         GuildToken(guild).incrementGauge(term, guildAmount);
```
## [L-04] Functions expected to be called
Below is expected to be called by the users. However, due to the network being served in, it might not be profitable to call the functions due to gas costs. Advised to implemented these functions at their superior functions as inlined.

```solidity
Contract: LendingTermOffboarding.sol

175:     function cleanup(address term) external whenNotPaused {
```

## [L-05] Bribing contract / whale domination
Offboarding is fairly much more easier as per the Quorum votes.
This opens the possibility of a bribing proxy contract or a whale to continously reach the Quorum and offboard the opponent term for every `POLL_DURATION_BLOCKS` intervals and the voting.





## [N-01] Incompatibility with EIP4337
The protocol is not compatible with account abstraction due to code size checking;

```solidity
Contract: ERC20MultiVotes.sol

155:     function _setContractExceedMaxDelegates(
156:         address account,
157:         bool canExceedMax
158:     ) internal {
159:         require(
160:  -->        !canExceedMax || account.code.length != 0,//@audit-info
161:             "ERC20MultiVotes: not a smart contract"
162:         ); // can only approve contracts
163: 
164:         canContractExceedMaxDelegates[account] = canExceedMax;
165: 
166:         emit CanContractExceedMaxDelegatesUpdate(account, canExceedMax);
167:     }
```

## [N-02] The protocol is not compatible with high decimal tokens
High decimal tokens should not be accepted as gauges/terms.
```solidity
Contract: SimplePSM.sol

65:     constructor(
66:         address _core,
67:         address _profitManager,
68:         address _credit,
69:         address _pegToken
70:     ) CoreRef(_core) {
71:         profitManager = _profitManager;
72:         credit = _credit;
73:         pegToken = _pegToken;
74: 
75:         uint256 decimals = uint256(ERC20(_pegToken).decimals());
76:  -->    decimalCorrection = 10 ** (18 - decimals);//@audit-info
77:     }
```
## [N-03] Blocktime consistency
The protocol referred the block time as in 13 seconds while it's [officially](https://ethereum.org/en/developers/docs/blocks/#block-time) 12 seconds and in reality it's somewhere 12 to 13 seconds.
