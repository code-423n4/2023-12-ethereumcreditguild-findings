# Onboarding process of new terms is very gas inefficient

GitHub:\
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L105-L212

A new term must first be deployed, and then a proposal process for the term starts. If the proposal fails, a lot of gas was wasted during the deployment. Saving the deployment parameters in a hash and propose the approval of that hash to be allowed to deploy for the given parameters could save a lot of gas.

Here we can see the onboarding process for a new term:

```solidity
function createTerm(address implementation, LendingTerm.LendingTermParams calldata params) external returns (address) {
  require(implementations[implementation], 'LendingTermOnboarding: invalid implementation');
  // must be an ERC20 (maybe, at least it prevents dumb input mistakes)
  (bool success, bytes memory returned) = params.collateralToken.call(
    abi.encodeWithSelector(IERC20.totalSupply.selector)
  );
  require(success && returned.length == 32, 'LendingTermOnboarding: invalid collateralToken');

  require(
    params.maxDebtPerCollateralToken != 0, // must be able to mint non-zero debt
    'LendingTermOnboarding: invalid maxDebtPerCollateralToken'
  );

  require(
    params.interestRate < 1e18, // interest rate [0, 100[% APR
    'LendingTermOnboarding: invalid interestRate'
  );

  require(
    // 31557601 comes from the constant LendingTerm.YEAR() + 1
    params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]
    'LendingTermOnboarding: invalid maxDelayBetweenPartialRepay'
  );

  require(
    params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%
    'LendingTermOnboarding: invalid minPartialRepayPercent'
  );

  require(
    params.openingFee <= 0.1e18, // open fee expected [0, 10]%
    'LendingTermOnboarding: invalid openingFee'
  );

  require(
    params.hardCap != 0, // non-zero hardcap
    'LendingTermOnboarding: invalid hardCap'
  );

  address term = Clones.clone(implementation);
  LendingTerm(term).initialize(
    address(core()),
    LendingTerm.LendingTermReferences({
      profitManager: profitManager,
      guildToken: guildToken,
      auctionHouse: auctionHouse,
      creditMinter: creditMinter,
      creditToken: creditToken
    }),
    params
  );
  created[term] = block.timestamp;
  emit TermCreated(block.timestamp, implementation, term, params);
  return term;
}

/// @notice Propose the onboarding of a term
function proposeOnboard(address term) external whenNotPaused returns (uint256 proposalId) {
  // Check that the term has been created by this factory
  require(created[term] != 0, 'LendingTermOnboarding: invalid term');

  // Check that the term was not subject to an onboard vote recently
  require(
    lastProposal[term] + MIN_DELAY_BETWEEN_PROPOSALS < block.timestamp,
    'LendingTermOnboarding: recently proposed'
  );
  lastProposal[term] = block.timestamp;

  // Check that the term is not already active
  // note that terms that have been offboarded in the past can be re-onboarded
  // and won't fail this check. This is intentional, because some terms might be offboarded
  // due to specific market conditions, and it might be desirable to re-onboard them
  // at a later date.
  bool isGauge = GuildToken(guildToken).isGauge(term);
  require(!isGauge, 'LendingTermOnboarding: active term');

  // Generate calldata for the proposal
  (
    address[] memory targets,
    uint256[] memory values,
    bytes[] memory calldatas,
    string memory description
  ) = getOnboardProposeArgs(term);

  // propose
  return Governor.propose(targets, values, calldatas, description);
}

```
