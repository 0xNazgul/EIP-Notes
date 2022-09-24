# EIP-1559: Fee market change for ETH 1.0 chain
A new TX pricing mechanism that includes fixed-per-block network fee that is burned and dynamically expands/contracts block sizes to deal with transient congestion.
* Format: `0x02 || rlp([chain_id, nonce, max_priority_fee_per_gas, max_fee_per_gas, gas_limit, destination, amount, data, access_list, signature_y_parity, signature_r, signature_s])`

## Motivation
Ethereum historically priced TX fees using a simple auction mechanism (user sends TX with bids(gas) miners order by bids). This leads to inefficiency:
* Mismatch between volatility of TX fee levels and social cost of TXs: bids to include TXs on mature blockchains that have enough usage so that blocks are full are volatile.
* Needless delays for users: With both per-block gas limit and volatility in TX volume, TXs are forced to wait for several blocks making it socially unproductive.
* Inefficiencies of first price auctions: TX senders publish a TX with a bid a maximum fee, miners choose the highest paying TX.This mechanism is known to be inefficient and so complex fee estimation algorithms are needed. But those too can end up not working well leading to fee overpayment.
* Instability of blockchains with no block reward: In the long run blockchains where there is no issuance at present intend to switch to rewarding miners entirely through fees. BUT there are issues with this that lead to instability, incentivizing mining sister blocks to steal TX fees (Selfish mining attacks).

What this will do is start a base fee amount to adjust up and down based on how congested the network is. Because base fee changes are constrained this allows for predictability. So wallets can auto-set gas fees for users reliably. 
* Miners only get to keep priority fee. Base fee is always burned. This burn counterbalances Ethereum inflation while still giving the block reward and priority fee to miners. 

## Specification
No point in coping over the whole implementation so refer to the [original](https://eips.ethereum.org/EIPS/eip-1559#specification)

## Security Considerations
* Increased Max Block Size/Complexity: This could cause problems if miners are unable to process a block fast enough forcing an empty block. Only an issue in short term size burst but some clients may not be able to handle this and clients to try their best to do so.
* Transaction Ordering: With majority of users not trying to compete with fees, ordering now depends on the clients internal implementation details as how they store the TXs in memory. One should sort those with the same fee based on time received to protect the network from spamming attacks. 
* Miners Mining Empty Blocks: A miner could mine empty blocks to lower base fee and then proceed to mine half full blocks and revert to sorting TXs by priority fee. Although this would not be profitable for that miner unless one was in control of `>50%` of hashing power(but at that point would make more money from double spend attack). 
	* If a miner pulled this off a simple increase in the elasticity multiplier to require more hashing power before the attack can even be profitable.
* ETH Burn Precludes Fixed Supply: With burning there is no guaranteed fixed Ether supply. Which could lead to instability in the long run for ETH will no longer be constant. There is no way to tell how much of an economic issue this could be so it does cause core developers to lose some control over Ether's long term quantity.