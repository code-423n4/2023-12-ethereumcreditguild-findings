## [L-01] Reorg issue affects the clone(), should use the cloneDeterministic() when creating new terms

When creating a term, the [`LendingTermOnBoarding::createTerm()` function](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L105-L168) uses the `clone()` instead of the `cloneDeterministic()` to prevent issues if a reorg chain occurs.

1. Alice creates a new Term

2. Bob notices the ongoing reorg (or just does it by coincidence), and creates a new Term

3. Alice's term gets votes.

4. Reorg is resolved in Bob's favor, thus, Bob's term gets the votes that were meant to be for Alice's term

5. If enough votes for Bob's term, Bob could execute the proposal and start borrowing on that term, which could use worthless collateral or a configuration such that can allow Bob to borrow at his convenience

**Fix:**
- Hash the `msg.sender` alongside the parameters of the `params` variable to generate a `salt`, and instead of using the `clone()`, opt to use the `cloneDeterministic()` and forward the new generated salt to the `cloneDeterminstic()`. In this way, even if a reorg occurs, the terms would be deployed a deterministic address.
```
function createTerm(
    address implementation,
    LendingTerm.LendingTermParams calldata params
) external returns (address) {
    ...

    //@audit-info => i.e. hashing the params of the term and the caller!
+   bytes32 salt = keeckak256(params.param1,params.param2,params.param3.....,msg.sender);

-   address term = Clones.clone(implementation);
+   address term = Clones.cloneDeterministic(implementation,salt);
    ...
}
```

---
## [L-02] Not refunding the excess native token that is sent when calling the emergencyAction()

The [`CoreRef::emergencyAction() function`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L87-L107) allows to execute arbitrary calls, it also allows to send the native token in each of these calls. The problem is that it does not return the excess native token that was sent when the function was called and the total native token that was spent.

> CoreRef.sol
```solidity
function emergencyAction(
    Call[] calldata calls
)
    external
    payable
    onlyCoreRole(CoreRoles.GOVERNOR)
    returns (bytes[] memory returnData)
{
    returnData = new bytes[](calls.length);
    for (uint256 i = 0; i < calls.length; i++) {
        address payable target = payable(calls[i].target);
        uint256 value = calls[i].value;
        bytes calldata callData = calls[i].callData;

        //@audit-info => sends the specified amount of native token in the value array
        (bool success, bytes memory returned) = target.call{value: value}(
            callData
        );
        require(success, "CoreRef: underlying call reverted");
        returnData[i] = returned;
    }
    //@audit-issue => Not checking if there was any unspent native token to be returned after the execution is completed
}
```

**Fix:**
- add up all the individual `calls.value` and at the end, verify if the `msg.value` is equals to the sum of all the `calls.value`, if not, returns the unspent native token to the caller

---
## [L-03] voting period will vary depending on the chain where contracts are deployed, in some chains the number of blocks for the voting period will be less and in others more!
- The value assigned to the votingPeriod is a number of blocks that would be in 1 year assuming that each block is produced ~ every 13 seconds. The problem is that depending the chain where the contracts are deployed, the block production might vary from the assumption that each block is produced ~ every 13 seconds. 
> GuildVetoGovernol.sol
```solidity
function votingPeriod() public pure override returns (uint256) {
    //@audit-issue => voting period will vary depending on the chain where contracts are deployed, in some chains the number of blocks for the voting period will be less and in others more!
    return 2425847; // ~1 year with 1 block every 13s
}
```

- Also, there is a comment in the `AuctionHouse contract` for the `auctionDuration` that assumes that blocks are produced every 13 seconds. Take special care when computing the value that will be assigned to that variable. The block production is not fixated to 13s on all chains
> AuctionHouse.sol
```solidity
/// @notice maximum duration of auctions, in seconds.
/// with a midpoint of 650 (10m50s) and an auction duration of 30min, and a block every
/// 13s, first phase will last around 50 blocks and each block will offer an additional
/// 1/(650/13)=2% of the collateral during the first phase. During the second phase,
/// every block will ask 1/((1800-650)/13)=1.13% less CREDIT in each block.
uint256 public immutable auctionDuration;
```

**Fix:**
- From what it looks like this function is not currently used, but if in the future is meant to be implement with a new feature, just take into account this information and adjust accordingly the number of blocks per year depending on the chain where the contracts are deployed.

---
## [L-04] Not pausing borrows when a loan is forgiven because no one bided on the loan's auction
- If a loan is auctioned and no one bidded on the auction, the loan can be forgiven and all the debt will be accrued as bad debt for the market, the collateral will be left in the contract. The problem is if an auction has reached this point and the term is still active, new borrows can still happen with a collateral that is worth close to nothing. If the borrows are not paused when a loan is forgiven under this circunstances, borrowers can open new loans putting at risk the market becasue those new loans will most likely default and generate more bad debt which will end up affecting the creditMultiplier and impacting the CreditToken holders.

> AuctionHouse.sol
```solidity
function forgive(bytes32 loanId) external {
    ...
    LendingTerm(_lendingTerm).onBid(
        loanId,
        msg.sender,
        0, // collateralToBorrower
        0, // collateralToBidder
        0 // creditFromBidder
    );

    //@audit-issue => auction is closed and all debt is accrued as bad debt, but the term is not paused!
    ...
}
```

**Fix:**
-  set hardcap to 0 in the associated LendingTerm contract to prevent new borrows

---
## [L-05] PSM won't work for tokens with more than 18 decimals
- There are some tokens that used more than 18 decimals, the PSM module won't work for any of these tokens. When setting up the `decimalCorrection` in the constructor, the operation will underflow and tx will always revert.

> SimplePSM.sol
```solidity
constructor(
    ...
) CoreRef(_core) {
    ...

    //@audit-info => Gets the decimals of the PegToken
    uint256 decimals = uint256(ERC20(_pegToken).decimals());

    //@audit-issue => PSM won't work for tokens with more than 18 decimals!
    decimalCorrection = 10 ** (18 - decimals);
}
```

**Fix:**
- I personally don't think this needs a fix, but rather just document it and let know users that PSM works only with tokens that have 18 decimals or lower.

---
## [L-06] As the creditMultiplier changes, the monetary cost of the minimum amount of CreditTokens to stake in a term will decrease
- When staking CreditTokens in the SurplusGuildMinter contract it is enforced that the amount to be staked is greater than a minimum value. The potential downfall is that as the creditMultiplier shrinks because bad debt, the monetary cost to stake the minimum amount will decrease. Example, if MIN_STAKE was 50 CreditTokens, and each CreditToken used to be worth 1USD, that means that the minimum monetary cost to stake was 50USD, if creditMultiplier shrinks and now each CreditToken is worth .8 USD, now, the monetary cost to stake would be 40USD.

> SurplusGuildMinter.sol
```solidity
function stake(address term, uint256 amount) external whenNotPaused {
    ...
    require(amount >= MIN_STAKE, "SurplusGuildMinter: min stake");
    ...
}
```
**Fix:**
- If it is considered that the stake should cost the same for everybody at any time, make sure to normalize the `MIN_STAKE` with the current value of the `creditMultiplier`. If the monetary cost doesn't matter, this could just be acknowledge and documented.

--- 
## [L-07] if exiting and entering during the distribution period, there could be a slight rounding issue that would cause a slight loss compared to the full amount of rewards to be distributed.
When a distribution is active, if users enters and exits rebase consistently across the distribution period, at the end of the distribution, the final targetValue will suffer a slight rounding issue (down) from the total rewards that should've been distributed. The loss is in the weis, not a really big impact.

I coded a PoC where this issue is demonstrated. Let's add the below function to the [`ERC20RebaseDistributor.t.sol test file`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/test/unit/tokens/ERC20RebaseDistributor.t.sol)
- Make sure to run two times this poc (See comments in the PoC's code)
`forge test --match-test testLossPrecisionPoC -vvv`

```solidity
...
import "@test/forge-std/src/console2.sol";

contract ERC20RebaseDistributorUnitTest is Test {
    ...

    function testLossPrecisionPoC() public {
        // token.mint(address(this), 100e18);
        // token.enterRebase();

        token.mint(alice, 100e18);
        // assertEq(token.totalSupply(), 200e18);
        console2.log("rebasingSharePrice before enteringRebase: ", token.rebasingSharePrice());
        console2.log("rebasingsupply:", token.rebasingSupply());
        console2.log("====================");

        vm.prank(alice);
        token.enterRebase();

        assertEq(token.isRebasing(alice), true);
        // assertEq(token.rebasingSupply(), 200e18);

        // distribute
        token.mint(address(this), 1000e18);
        token.distribute(1000e18);
        console2.log("rebasingSharePrice after distribute: ", token.rebasingSharePrice());
        console2.log("rebasingsupply after distribute:", token.rebasingSupply());
        console2.log("====================");

        uint256 rebasingSharePriceBeforeBobby = token.rebasingSharePrice();
        uint256 rebasingSupplyBeforeBobby = token.rebasingSupply();

        token.mint(bobby, 10000e18);

        vm.prank(bobby);
        token.enterRebase();

        vm.prank(bobby);
        token.exitRebase();

        vm.prank(bobby);
        token.enterRebase();
        console2.log("rebasingSharePrice after bobby reenteredRebase: ", token.rebasingSharePrice());
        console2.log("rebasingsupply:", token.rebasingSupply());
        console2.log("====================");

        //@audit-info => Run the PoC one time with this line uncommented and see the output
        vm.warp(block.timestamp + token.DISTRIBUTION_PERIOD());

        //@audit-info => The second execution make sure to uncomment the below lines and comment the above line (the vm.warp() line....)
        // uint256 jumps = token.DISTRIBUTION_PERIOD() / 5;
        // vm.startPrank(bobby);
        // skip(jumps);
        // token.exitRebase();
        // skip(jumps);
        // token.enterRebase();
        // skip(jumps);
        // token.exitRebase();
        // skip(jumps);
        // token.enterRebase();
        // skip(jumps);
        // token.exitRebase();
        // skip(jumps);
        // token.enterRebase();
        // skip(jumps);
        // token.exitRebase();
        //  skip(jumps);
        // token.enterRebase();
        // skip(jumps);
        // token.exitRebase();
        //  skip(jumps);
        // token.enterRebase();
        // skip(jumps);
        // token.exitRebase();
        //  skip(jumps);
        // token.enterRebase();
        // skip(jumps);
        // token.exitRebase();
        //  skip(jumps);
        // token.enterRebase();
        // vm.stopPrank();
        // console2.log("rebasingSharePrice after bobby plays around: ", token.rebasingSharePrice());
        // console2.log("rebasingsupply:", token.rebasingSupply());
        // console2.log("====================");

        vm.prank(alice);
        token.exitRebase();
        vm.prank(bobby);
        token.exitRebase();
        console2.log("rebasingSharePrice after exiting rebase: ", token.rebasingSharePrice());
        console2.log("rebasingsupply after exiting rebase:", token.rebasingSupply());
        console2.log("====================");

    }
    ...
}
```

I think it's worth reporting this, though i did not find a way to exploit it or a side effect caused by the rounding issue.

**Fix:**
- I'm unclear if this issue actually requires a fix, but it is definitely worth documenting this behavior.

### Time spent:
1 hours