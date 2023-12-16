## [L-01] `RateLimitedV2` initializes `lastBufferUsedTime` as uint32 in the second storage slot.

`Type(uint32).max` is equal to 4_294_967_295 which converted into human-readable is 7 February 2106 6:28:15. Consider changing it to uint40, `type(uint40).max` (1_099_511_627_775) gives a date that we can't comprehend. 

```diff
    /// ------------- Second Storage Slot -------------

    /// @notice the last time the buffer was used by the contract
--  uint32 public lastBufferUsedTime;
++  uint40 public lastBufferUsedTime;
    /// @notice the buffer at the timestamp of lastBufferUsedTime
--  uint224 public bufferStored;
++  uint216 public bufferStored;
```
Further adjust bufferStored usage throughout code to match the new uint216 type.