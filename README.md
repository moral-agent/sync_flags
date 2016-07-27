# Sync Flags

The idea presented here could have the following benefits:
 
1. Improve mining decentralization
2. Reduce variance in mining profitability
3. Reduce or eliminate SPV mined blocks
4. Reduce or eliminate empty blocks, smoothing out resource usage
5. Reduce or eliminate the latency bottleneck on throughput
6. Make transaction stuffing by miners be either obvious or costly
7. Gives miners something to do while they wait for attractive transactions to appear
8. Can be easily done with a soft fork
 
#Basic idea:
 
Ideally, all miners would begin hashing the next block at exactly the same time. Miners with a head start are more profitable, and the techniques that help miners receive and validate blocks quickly create centralization pressure.
 
What if there was something that acted like the starting flag at a race, which could suddenly wave and cause all of the miners to simultaneously begin hashing the next block?
 
#Implementation:
 
Let a sync flag be a message consisting of:
 
1. Hash of the previous block.
2. Donation address
3. Nonce
 
This tiny message could propagate through the network at maximum speed. If miners had to include the hash of this flag in the next block, then all miners wait for this flag, and when it suddenly spread through the network, all miners could simultaneously begin hashing the next block.
 
The hash of the sync flag is a proof of work with the same difficulty as normal bitcoin blocks. To fund this proof of work, the protocol is modified to demand that the 10th, 11th, 12th, 13th, 14th blocks produced after the sync flag must each donate 10% of their block reward to the address supplied by the sync block.
 
 
Illustration 1: https://s32.postimg.org/wzg0hs8lx/sync_flag.png)
 
Illustration 2: https://s32.postimg.org/vc5y9yz4l/sync_flag2.png
 
 
#Explanation of reasons:
 
**Improve mining decentralization**
 
One factor driving centralization is the imperative miners have to achieve low latency in receiving and validating blocks. To achieve low latency, it helps a lot if you have expensive low-latency internet connections, curated network topologies, and large pools that have a plausible chance of finding consecutive blocks. If miners are expected (or forced) to validate a block prior to mining on top of it, the rational end game would be to outsource the validation step to a trusted third party specialist who can choose optimal locations on the globe to serve their (multiple?) mining pool clients. These are all less decentralized than the mining situation Satoshi and others imagined.
 
**Reduce variance in mining revenue**
 
Currently, there are about 144 opportunities per day for a pool or solo miner to see any revenue at all. With sync flags, that number doubles to 288. This variance is one factor causing solo miners to group into pools, and large pools to be more attractive than small pools.
 
**Reduce or eliminate SPV mined blocks**
 
One way miners have sought to make full-block-transmission-and-validation-latency irrelevant has been through "SPV" mining or "Head-first" mining. There is some evidence that these techniques may be widely used, and that badgering the miners about it is an ineffective strategy to stop them.
 
In SPV mining, a miner would simply accept any block header that shows the correct proof of work. All other validation is entrusted to other miners. This practice is quite dangerous as the SPV miners can wander off on some invalid chain, taking SPV nodes with them. If this occurs during a soft fork, these blind miners can also fool unupgraded fully validating nodes into following them.
 
"Head-first" mining means that miners start hashing as soon as they receive the block header with the correct POW, but they simultaneously validate the block, and abandon it if is not valid. I consider this to be pretty safe, as it strictly limits the length of an invalid chain that can result from mining without validating. However, "Head-first" mining can plausibly generate 2 or 3 confirmations of an invalid block. It would be nice if such confirmations did not happen.
 
The sync flag technique is similar to head-first mining, but rather than hashing the next block while they wait for transmission and validation of the prior block, they hash the sync flag. Nodes can differentiate between sync flags and blocks, and can ignore sync flags when counting confirmations.
 
**Reduce or eliminate empty blocks, smoothing out resource usage**
 
Empty blocks are another consequence of SPV or Headfirst mining, because the miner cannot safely include any transactions in the block they are hashing until they have validated the prior block. By delaying the start of hashing the next block until after validation, miners would not have this reason to mine empty blocks.
 
**Reduce or eliminate the latency bottleneck on throughput**
 
Centralization pressure due to latency issues has been a major preoccupation over the last year. If latency mattered much less, it could represent a scalability improvement that could enable higher throughput.
 
**Make transaction stuffing by miners be either obvious or costly**
 
Currently, the entire block reward goes to the miner who mines it. One unfortunate consequence of this is that it does not cost the miner anything to covertly stuff the block with transactions. These transactions would pay fees and be indistinguishable from ordinary transactions, but the fees are paid by the miner and then immediately returned to the miner.
 
With sync flags, the miner must share 50% of the transaction fee, distributing it equally to the sync blocks 10, 11, 12, 13 and 14 blocks prior. This means that if the miner gives the transactions a normal looking fee, they will incur a cost that will be paid to the sync flag. If the miner wants to avoid this, they must give their stuffing transactions a zero fee, which provides evidence that they are stuffing.
 
Also, when miners stuff with transactions using a zero fee, they cannot manipulate the perception of how much fee it takes to get into a block.
 
Miners destribute the donation equally to 5 different sync blocks to avoid two kinds of attack that are easiest to visualize if you imagine the alternative that the miners donate to only a single sync block. In this scheme, they can adjust their bias their mining strategy depending on who mined the sync block. If the sync block belongs to the miner, the anti-stuffing incentive described above would not be effective. If the sync block belongs to a weak competitor, the miner might attack by producing a block with no transactions, so as to reduce the profitability of their competitor in the hope of driving them out of business.
 
**Gives miners something to do while they wait for attractive transactions to appear**
 
From the Montreal scaling workshop last year, we have [this talk](https://scalingbitcoin.org/montreal2015/presentations/Day1/13-miles-carlsten-Mind-the-Gap.pdf) which worried that as the block subsidy reduced and transactions became a more important fraction of miner revenue, it would be rational for miners to turn off their mining equipment for a "gap" phase after a block is found, to allow time to pass as more lucrative transactions entered the mempool.
 
I don't know whether this will actually happen. The presence of a suitable backlog of transactions would help prevent this dynamic from emerging. But if such idling behavior was the optima mining strategy, it could create a serious vulnerability. Idle hands are the devil's workshop as the saying goes, and idle miners represent a pool of inert hashpower that is available to rent for all kinds of destabilizing purposes. It would be better to put those miners to profitable work mining a sync flag while they wait.
 
Also, this creates a more efficient price discovery mechanism for transactions, because you allow transactions paying high fees time to arrive to the marketplace, rather than take whatever anyone is offering because all the "good" transactions got gobbled up in the prior block.
 
**Can be easily done with a soft fork**
 
Although a hard fork would be more efficient, sync flags could be easily implemented using a soft fork by introducing the following rule:
 
Every block must include a transaction which pays 10% of the block reward to the address given by the 10th, 11th, 12th, 13th and 14th previous sync flags, and commits to the hash of the 1st previous sync flag.

#Comment on selfish mining:

Making the sync flag POW target the same as the block reward target is a nod to selfish mining. I have not done any simulations to test this, but it seems that if the sync flag had a lower POW than the block it would be rational for a miner who has just found a block to hide it and begin mining the flag. Insofar as the flag is easier to mine than a block, they might try to mine it while everyone else is still laboring on the block.

#Comment on invalid block spam:

It is possible for a miner to (intentionally or not) attack other miners by producing a block header with the correct proof of work, but which does not contain valid transactions. Defensive miners must decide whether to hash before validating the block or after validating the block. This is a dilemma because both strategies have a downside. If they fully validate the block first, then they will tend to be at a disadvantage in finding the next block compared with miners who hash immediately, and especially compared with the author of the previous block, who already knows whether or not the block is valid. If they hash immediately, then whenever they receive an invalid block, they will waste their hashpower on it until they can learn that it is invalid.

This problem exists in the current protocol, but the problem has somewhat different attributes in the sync flag model.

Problems with the sync flag protocol:

1. Invalid blocks are half as expensive to create. They would remain quite expensive, but perhaps this discount would make the attack more feasible.
2. Because sync flags are produced in 5 minutes instead of 10, miners have less time to waste by validating blocks, and therefore would be more likely to choose the "head first" mining alternative.

Advantages of the sync flag protocol:
1. Miners who have not validated the prior block cannot include any transactions when they "head first" mine. If they are mining the sync flag, this is not a disadvantage, since no one can include transactions.
2. The production of a sync flag on top of an invalid block need not constitute a confirmation from the point of view of SPV nodes. These nodes can reason that a sync flag may have been produced before validation was complete, and only regard blocks as confirmations.

One avenue to investigate for mitigating the risk due to invalid block spam might be punishment. For example, if a means could be devised for a miner to put  up a security deposit that could be collected if the block were invalid, then the penalty for producing this spam could raised if necessary. Fortunately this kind of spam does not currently appear to exist -- possibly because it is so expensive and there are not strong incentives to do it.