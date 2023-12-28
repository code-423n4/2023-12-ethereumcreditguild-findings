### Cache Memory variable
```
function propose(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        string memory description
    ) public virtual override returns (uint256) {
        address proposer = _msgSender();
        require(_isValidDescriptionForProposer(proposer, description), "Governor: proposer restricted");

        uint256 currentTimepoint = clock();
        require(
            getVotes(proposer, currentTimepoint - 1) >= proposalThreshold(),
            "Governor: proposer votes below proposal threshold"
        );

        uint256 proposalId = hashProposal(targets, values, calldatas, keccak256(bytes(description)));
        // @audit : gas otmization chache length of traget.length
       @ require(targets.length == values.length, "Governor: invalid proposal length");
        @ require(targets.length == calldatas.length, "Governor: invalid proposal length");
       @ require(targets.length > 0, "Governor: empty proposal");
        

       
    }
```
Can be changed to this:
```
uint256 _targets = targets.length;
// @audit : gas otmization chache length of traget.length
       @ require(_targets == values.length, "Governor: invalid proposal length");
        @ require(_targets == calldatas.length, "Governor: invalid proposal length");
       @ require(_targets > 0, "Governor: empty proposal");
```
