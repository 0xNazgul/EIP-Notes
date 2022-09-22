# EIP-1283: Net gas metering for SSTORE without dirty maps
Proposes net gas metering changes for `SSTORE`, enabling new usages for contract sotrage and reducing excessive gas costs where it doesn't match how most implementation works.


## Motivation
A way for gas metering on `SSTORE` using information that is more universally available to most implementations and require as little change in implementation structures.
* Storage slot’s original value.
* Storage slot’s current value.
* Refund counter.

benefactors of this gas reduction:
* Subsequent storage write operations in the same call frame. reentrancy locks, same-contract multi-send etc.
* Exchange storage info between sub call frame and parent call frame, where this info doesn't need to be persistent outside of a TX (includes sub-frame errors and message passing, etc).

## Specification
Terms:
* Storage slot’s original value: This is the value of the storage if a reversion happens on the current transaction.
* Storage slot’s current value: This is the value of the storage before `SSTORE` operation happens.
* Storage slot’s new value: This is the value of the storage after `SSTORE` operation happens.

Replace `SSTORE` opcode gas cost calculation (including refunds) with:
* If current value = new value (no-op), 200 gas is deducted.
* If current value != new value
	* If original value = current value 
		* If original value is 0, 20_000 gas is deducted.
		* Otherwise, 5000 gas is deducted. If new value is 0, add 15000 gas to refund counter.
	* If original value != current value (this storage slot is dirty), 200 gas is deducted. Apply both of the following clauses.
		* If original value != 0
			* If current value is 0 (also means that new value != 0), remove 15_000 gas from refund counter. We can prove that refund counter will never go below 0.
			* If new value is 0 (also means that current value != 0), add 15_000 gas to refund counter.
		* If original value equals new value (this storage slot is reset)
			* If original value is 0, add 19_800 gas to refund counter.
			* Otherwise, add 4_800 gas to refund counter.

Refund counter works as before and it is limited to half of the gas consumed. On TX level refund counter will never go below 0.
* If an implementation uses “TX level” refund counter (refund is checkpointed at each call frame), then the refund counter continues to be unsigned.
* If an implementation uses “execution-frame level” refund counter (a new refund counter is created at each call frame, and then merged back to parent when the call frame finishes), then the refund counter needs to be changed to signed at internal calls, a child refund can go below 0.

## Explanation
The new gas cost scheme for `SSTORE` is divided into three different types:
* `No-op`: the VM does not need to do anything. This is the case if current value = new value.
* `Fresh`: this storage slot has not been changed, or has been reset to its original value. This is the case if current value does not equal new value, and original value equals current value.
* `Dirty`: this storage slot has already been changed. This is the case if current value does not equal new value, and original value does not equal current value.

All initial (not-No-op) `SSTORE` on a particular storage slot starts with `Fresh`. It will become `Dirty` if the value has changed. Going from `Fresh -> Dirty` has the gas cost the same. A `Dirty` storage slot can be rest back to `Fresh` via `SSTORE` and trigger a refund.
* When a storage slot remains `Dirty` it charges 200 gas. This case needs to keep track of `R_sclear` refunds. If it already issued the refund but it no longer applies then it removes this refund from the refund counter.
* If it didn't issue the refund but it is now applied then it adds this refund to the refund counter. 
## State Transition

![](https://eips.ethereum.org/assets/eip-1283/state.png)