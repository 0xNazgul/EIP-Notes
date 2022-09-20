# EIP-658: Embedding transaction status code in receipts
replaces the intermediate state root field of the receipt with a status code indicating if the top-level call succeeded or failed.

## Motivation
Because of EIP-140 it is not possible for users to assume a TX failed if and only if it consumed all gas. With this there is no clear for callers to tell if the TX succeeded and the state changes were applied.
* Full nodes can provide RPCs to get a TX return status and value 
* Fast nodes can only do this for nodes after their pivot point
* Light nodes cannot do this

This is to propose to replace the intermediate state root, already obsoleted by EIP-98 with the return status. 
* This allows both callers to determine success status and remedies the omission of return data from the receipt.

## Specification
The intermediate state root is replaced by a status code, 0 indicating failure (due to any operation that can cause the transaction or top-level call to revert) and 1 indicating success.
