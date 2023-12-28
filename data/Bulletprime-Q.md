|---|---|---|
| [L-01]| Unfair Voting Mehanism | 1 |
|[L-02] | Poor Parameter Validation | 1 |
|[L-03] | External Calls | 1 |

DETAILS

[L-01] Unfair Voting Mehanism.
The `LendingTermOffboarding` contract lies the `supportOffboard` function, This function is used to support a removal poll, and it allows users to vote for the removal of a lending term. However, there's an issue in the way the voting weight is calculated.

```
solidity
function supportOffboard(
    uint256 snapshotBlock,
    address term
) external whenNotPaused {
    require(
        block.number <= snapshotBlock + POLL_DURATION_BLOCKS,
        "LendingTermOffboarding: poll expired"
    );
    uint256 _weight = polls[snapshotBlock][term];
    require(_weight != 0, "LendingTermOffboarding: poll not found");
    uint256 userWeight = GuildToken(guildToken).getPastVotes(
        msg.sender,
        snapshotBlock
    );
    require(userWeight != 0, "LendingTermOffboarding: zero weight");
    require(
        userPollVotes[msg.sender][snapshotBlock][term] == 0,
        "LendingTermOffboarding: already voted"
    );

    userPollVotes[msg.sender][snapshotBlock][term] = userWeight;
    polls[snapshotBlock][term] = _weight + userWeight;
    if (_weight + userWeight >= quorum) {
        canOffboard[term] = true;
    }
    emit OffboardSupport(
        block.timestamp,
        term,
        snapshotBlock,
        msg.sender,
        userWeight
    );
}
```
In this function, the user's voting weight is calculated based on their past votes and added to the poll's current voting power. If this combined weight meets or exceeds the quorum, the term can be offboarded.
The exploit here is that an attacker could potentially manipulate the voting weight to meet the quorum and offboard a term. This could be done by creating multiple accounts and using them to vote for the removal of a term. The `userPollVotes` mapping is used to prevent a user from voting multiple times, but an attacker with multiple accounts could potentially bypass this check.Thereby skewing the results. To mitigate this, you could introduce a mechanism to track the number of votes per account, closely monitor any unusual activity, such as a sudden increase in the voting power of a poll. Additionally, the contract could be modified to limit the amount of voting power a single user can have in a poll, or to impose a cooldown period between votes. An example of how you could modify the `supportOffboard` function to implement this,

```
solidity
mapping(address => uint256) private userVoteCount;

function supportOffboard(
   uint256 snapshotBlock,
   address term
) external whenNotPaused {
   require(
       block.number <= snapshotBlock + POLL_DURATION_BLOCKS,
       "LendingTermOffboarding: poll expired"
   );
   uint256 _weight = polls[snapshotBlock][term];
   require(_weight != 0, "LendingTermOffboarding: poll not found");
   uint256 userWeight = GuildToken(guildToken).getPastVotes(
       msg.sender,
       snapshotBlock
   );
   require(userWeight != 0, "LendingTermOffboarding: zero weight");
   require(
       userPollVotes[msg.sender][snapshotBlock][term] == 0,
       "LendingTermOffboarding: already voted"
   );
   require(
       userVoteCount[msg.sender] < MAX_VOTES_PER_ACCOUNT,
       "LendingTermOffboarding: max votes exceeded"
   );

   userVoteCount[msg.sender]++;
   userPollVotes[msg.sender][snapshotBlock][term] = userWeight;
   polls[snapshotBlock][term] = _weight + userWeight;
   if (_weight + userWeight >= quorum) {
       canOffboard[term] = true;
   }
   emit OffboardSupport(
       block.timestamp,
       term,
       snapshotBlock,
       msg.sender,
       userWeight
   );
}
```
In this version, a `userVoteCount` mapping is introduced to keep track of the number of votes each account has made. Before allowing a vote, the function checks if the user has reached the maximum allowed number of votes (MAX_VOTES_PER_ACCOUNT). If they have, the function reverts. Otherwise, it increments the user's vote count and allows the vote.
This change should make the voting process more fair and less susceptible to manipulation by malicious actors.


[L-02]  Poor Parameter Validation 

The `createTerm` function in the `LendingTermOnboarding` contract there are several checks in place to validate the parameters passed to it. These checks ensure that the parameters are within acceptable ranges and meet certain conditions. Although these checks are based off assumptions, For instance, the `params.interestRate` is assumed to be a percentage less than 100%, and `params.maxDelayBetweenPartialRepay` is assumed to be less than a certain constant value. If these assumptions are incorrect, it could lead to unexpected behavior.

```
solidity
function createTerm(
  address implementation,
  LendingTerm.LendingTermParams calldata params
) external returns (address) {
  require(
      implementations[implementation],
      "LendingTermOnboarding: invalid implementation"
  );
  // must be an ERC20 (maybe, at least it prevents dumb input mistakes)
  (bool success, bytes memory returned) = params.collateralToken.call(
      abi.encodeWithSelector(IERC20.totalSupply.selector)
  );
  require(
      success && returned.length == 32,
      "LendingTermOnboarding: invalid collateralToken"
  );

}

```
To mitigate these issues, you could add additional checks to ensure that the parameters meet the correct conditions. For example, you could add a check to ensure that `params.interestRate` is a positive number. 



L3 External calls 
The `createTerm` function in the `LendingTermOnboarding` contract, the function makes an external call to `params.collateralToken.call()`. Although the function checks the return value of this call, it doesn't check whether the call itself was successful. If the call fails for some reason, the function could behave incorrectly.

```
solidity
(bool success, bytes memory returned) = params.collateralToken.call(
  abi.encodeWithSelector(IERC20.totalSupply.selector)
);
require(
  success && returned.length == 32,
  "LendingTermOnboarding: invalid collateralToken"
);

```
consider adding error handling to deal with failed external calls.

```
function createTerm(
   address implementation,
   LendingTerm.LendingTermParams calldata params
) external returns (address) {
   require(
       implementations[implementation],
       "LendingTermOnboarding: invalid implementation"
   );
   // Additional check for interestRate
   require(
       params.interestRate > 0,
       "LendingTermOnboarding: invalid interestRate"
   );
   // must be an ERC20 (maybe, at least it prevents dumb input mistakes)
   (bool success, bytes memory returned) = params.collateralToken.call(
       abi.encodeWithSelector(IERC20.totalSupply.selector)
   );
   require(
       success && returned.length == 32,
       "LendingTermOnboarding: invalid collateralToken"
   );
  
}
```
Additionally, you could add a try-catch statement around the external call to handle any errors that might occur.

```
try params.collateralToken.call(abi.encodeWithSelector(IERC20.totalSupply.selector)) {
   // Call was successful
} catch Error(string memory reason) {
   // Handle error
} catch (bytes memory /*lowLevelData*/) {
   // Handle error
}
```
These modifications should help make the function more robust against potential attacks.
