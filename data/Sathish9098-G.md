# GAS OPTIMIZATION

##

## [G-1] ``dropperAdmin`` and ``quorumVotesBPS`` state variables can be packed within same slot : Saves ``2000 GAS`` , ``1 SLOT ``

Use ``uint96`` for ``quorumVotesBPS`` instead of ``uint256`` is indeed appropriate and efficient, given the constraint that ``quorumVotesBPS`` will not exceed ``6,000 `` (MAX_QUORUM_VOTES_BPS).

This constraint is enforced by the check require(_cultureIndexParams.quorumVotesBPS <= MAX_QUORUM_VOTES_BPS, "invalid quorum bps");, ensuring that the value of quorumVotesBPS remains within the bounds that uint96 can comfortably handle.


```diff
FILE: 2023-12-revolutionprotocol/packages/revolution/src/CultureIndex.sol

  /// @notice The basis point number of votes in support of a art piece required in order for a quorum to be reached and for an art piece to be dropped.
-    uint256 public quorumVotesBPS;

    /// @notice The name of the culture index
    string public name;

    /// @notice A description of the culture index - can include rules or guidelines
    string public description;

    // The list of all pieces
    mapping(uint256 => ArtPiece) public pieces;

    // The internal piece ID tracker
    uint256 public _currentPieceId;

    // The mapping of all votes for a piece
    mapping(uint256 => mapping(address => Vote)) public votes;

    // The total voting weight for a piece
    mapping(uint256 => uint256) public totalVoteWeights;

    // Constant for max number of creators
    uint256 public constant MAX_NUM_CREATORS = 100;

    // The address that is allowed to drop art pieces
    address public dropperAdmin;
+    uint96 public quorumVotesBPS;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/CultureIndex.sol#L53-L78

##

## [G-2] ``creatorRateBps`` and ``treasury`` , ``entropyRateBps``,``creatorsAddress`` state variables can be packed within same slot : Saves ``4000 GAS`` , ``2 SLOTs``


``creatorRateBps`` and ``entropyRateBps`` are guaranteed not to exceed 10,000 based on the checks in the setter functions (_entropyRateBps <= 10_000 and _creatorRateBps <= 10_000), then using uint96 for these variables instead of uint256 is a more efficient choice. 

```diff
FILE: 2023-12-revolutionprotocol/packages/revolution/src
/ERC20TokenEmitter.sol

 // treasury address to pay funds to
    address public treasury;
+    uint96 public creatorRateBps;

    // The token that is being minted.
    NontransferableERC20Votes public token;

    // The VRGDA contract
    VRGDAC public vrgdac;

    // solhint-disable-next-line not-rely-on-time
    uint256 public startTime;

    /**
     * @notice A running total of the amount of tokens emitted.
     */
    int256 public emittedTokenWad;

    // The split of the purchase that is reserved for the creator of the Verb in basis points
-    uint256 public creatorRateBps;

    // The split of (purchase proceeds * creatorRate) that is sent to the creator as ether in basis points
-    uint256 public entropyRateBps;
+    uint96 public entropyRateBps;

    // The account or contract to pay the creator reward to
    address public creatorsAddress;

```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/ERC20TokenEmitter.sol#L24-L48

##

## [G-3] ``creatorRateBps,verbs``,``minCreatorRateBps,erc20TokenEmitter``, ``entropyRateBps,WETH`` can be packed with same slot : Saves ``6000 GAS`` , ``3 SLOT``

creatorRateBps,minCreatorRateBps,entropyRateBps can be uint96 instead of uint256. As per setter function checks  ``_creatorRateBps >= minCreatorRateBps``, ``_creatorRateBps <= 10_000``,`` _entropyRateBps <= 10_000`` the values not exceeds 10000 . 

```diff
FILE: 2023-12-revolutionprotocol/packages/revolution/src
/AuctionHouse.sol

// The Verbs ERC721 token contract
    IVerbsToken public verbs;
+    uint96 public creatorRateBps;

    // The ERC20 governance token
    IERC20TokenEmitter public erc20TokenEmitter;
+    uint96 public minCreatorRateBps;

    // The address of the WETH contract
    address public WETH;
+    uint96 public entropyRateBps;

    // The minimum amount of time left in an auction after a new bid is created
    uint256 public timeBuffer;

    // The minimum price accepted in an auction
    uint256 public reservePrice;

    // The minimum percentage difference between the last bid amount and the current bid
    uint8 public minBidIncrementPercentage;

    // The split of the winning bid that is reserved for the creator of the Verb in basis points
-    uint256 public creatorRateBps;

    // The all time minimum split of the winning bid that is reserved for the creator of the Verb in basis points
-    uint256 public minCreatorRateBps;

    // The split of (auction proceeds * creatorRate) that is sent to the creator as ether in basis points
-    uint256 public entropyRateBps;


```
https://github.com/code-423n4/2023-12-revolutionprotocol/blob/d42cc62b873a1b2b44f57310f9d4bbfdd875e8d6/packages/revolution/src/AuctionHouse.sol#L47-L72






