1.Variables are created for one time usage and can be avoided.

File Name : CoreRef.sol

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


           (bool success, bytes memory returned) = target.call{value: value}(
               callData
           );
           require(success, "CoreRef: underlying call reverted");
           returnData[i] = returned;
       }
   }


Creation of variables  target, value, and callData can be avoided. 

The resulting code will be 

      for (uint256 i = 0; i < calls.length; i++) {


           (bool success, bytes memory returned) = payable(calls[i].target).call{value: calls[i].value}(
               calls[i].callData
           );
           require(success, "CoreRef: underlying call reverted");
           returnData[i] = returned;
       }

Note: Observed gas savings in remix. 

2.Use of variable name similar to solidity keyword: 
Calldata is an internal keyword used in solidity language. Using a variable name similar to keyword is not a good coding practice.
In coreRef.sol →bytes calldata callData
In GuildVetoGovernor.sol → bytes[] memory calldatas


