# EIP-2387 Muir Glacier
Ethereum has a consistent block time due to it's difficulty retargeting algorithm. When a block-time is higher than 20 seconds the difficulty is reduced and if lower than 10 seconds it is increased. Reaching an equilibrium of around 13-14 seconds. With all of this there is what they call a Difficulty Bomb or Ice Age.

It adds to the difficulty in a way that the retargeting mechanism will at some point not adapt to the increase. The Ice Age increments every `100,000` blocks and would have snowballed over time. 

The fork is meant to push this back as far as possible and give core devs time to fix these design problems. The solutions under consideration:
* Update the mechanism so that behavior is predictable
* Remove the Ice Age entirely

## Activation:
* Block >= 9,200,000 on the Ethereum mainnet
* Block >= 7,117,117 on the Ropsten testnet
* Block >= N/A on the Kovan testnet
* Block >= N/A on the Rinkeby testnet
* Block >= N/A on the Görli testnet

## Included EIPs
* [EIP-2384]()

# Rationale
*Original intention of the Ice Age*:
* At the time of upgrades, inhibit unintentional growth of the resulting branching forks leading up to Eth 2.0
* Encourage a prompt upgrade schedule
* Forces the community to come back into agreement repeatedly
* Is a check for the Core Devs in the case that a decision is made to freeze the code base of clients without the blessing of the community

NOTE: is meant to encourage core devs to upgrade along with the network. Preventing the case where sleeper forks remain dormant only to be resurrected. This is what Ethereum Classic has done.