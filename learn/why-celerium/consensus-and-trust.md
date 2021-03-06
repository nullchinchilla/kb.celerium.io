# Consensus and trust

We start our exploration of Celerium's features by looking at how nodes in Celerium come to agreement on the status of the network --- decentralized **consensus**, the foundation of any public blockchain's security. Celerium uses a variation of **bonded proof-of-stake** found in systems such as Tendermint, augmented with "auditors" which further decentralize trust.

## Oligarchy with a free press

Participants in Celerium are divided into three categories by their roles:

* **Stakeholders** record transactions into new blocks and confirm them using a Byzantine fault-tolerant consensus algorithm between themselves. They communicate with each other through a broadcast protocol which other nodes in the network never participate in. The set of stakeholders is determined on the blockchain by "staking" a cryptoasset; the staking process will be discussed in TODO. Stakeholders correspond to miners or validators in other systems.
* **Auditors** download newly created blocks from the stakeholders and gossip them between themselves, while storing a local copy of the blockchain. Anybody can join the network as a auditor, and they verify the new blocks decided by the stakeholders and check that the stakeholders never equivocate on the content of a given block height. Auditors roughly correspond to full nodes in other systems, although they have a more important role in Celerium's security.
* **Clients** are lightweight participants that query the network of auditors to access specific information in the blockchain, yet do not trust any particular auditor.

From this overview we can already see that the trust model of Celerium differs significantly both from that of traditional proof-of-work public blockchains like Bitcoin and from that of typical private blockchains, and is one of its major innovations. Celerium's trust can be summarized succinctly as an **"oligarchy with a free press"**, in contrast to private blockchains' "closed oligarchy/monarchy" or traditional public blockchains' "direct democracy".

## Stakeholders: the oligarchy

### Bonded proof-of-stake

In Celerium, a Byzantine-resistant fault tolerant algorithm is used between the stakeholders to establish consensus on the content of the blockchain. The stakeholders form an "oligarchy", as the majority of participants are not stakeholders, yet they decide the authoritative state of the network.

How does anybody become a stakeholder, and how is power distributed between the stakeholders? Celerium uses a variation on a classic technique known as bonded proof of stake, used in systems like Tendermint and Casper. We keep track of a special secondary currency on the blockchain known as the **lent**. Lents are traded freely alongside cels, the main cryptocurrency of Celerium, with a regulated supply of 1 lent per block \(1.05 million lents per year\). They can be thought of as "shares" in a decentralized corporation in charge of deciding new blocks.

{% hint style="info" %}
**Lent** comes from _lentus_, the Latin word for "slow", a reference to how lents must be frozen by stakeholders to establish voting rights.
{% endhint %}

In order to become a stakeholder, a holders of lents can **stake** at least 10,000 lents, locking them up for a fixed period of time \(500,000 blocks, or approximately 6 months\) to obtain voting rights in the consensus algorithm in proportion to the amount of lents staked. The central security assumption Celerium uses is that _at least 2/3 of the staked lents are in the hands of honest stakeholders_, which derives from a fundamental property of asynchronous Byzantine-fault-tolerant consensus.

### Rewards and slashing

How do we incentivize stakeholders to behave honestly? We use a carrot-and-stick approach commonly found in systems using bonded proof of stake: honest stakeholders earn **rewards** over time in proportion to the amount of lents they stake, while misbehaving stakeholders can have their entire stake **slashed** given evidence of misbehavior or **kicked** from being a stakeholder.

Rewards to stakeholders come from two sources: lent inflation and transaction fees. Stakeholders proposing new blocks earn rewards of 1 newly minted lent per block, just like how Bitcoin miners earn a fixed per-block reward. Lent inflation also serves a double purpose of encouraging staking of lents rather than holding them passively.

Transaction fees, denominated in cels, are imposed on every transaction on the network, and these fees also go entirely to the stakeholders as an additional source of income. The details of exactly how much transaction fees are charged and exactly how they are paid out to stakeholders are a little complicated, as Celerium uses an idiosyncratic transaction fee model, different from that of other cryptocurrencies, in order to make it easier for clients to calculate appropriate fees. We discuss transaction fees more in the ~~standalone document on fees~~; for now a reasonable approximation is that stakeholders receive fees in proportion to the volume of transactions on the network.

In addition to rewards that incentivize contributing to the network as a stakeholder, we disincentivize misbehavior by two separate mechanisms: slashing and kicking.

Slashing is used to punish cryptographically provable misbehavior. The last step of our Byzantine-fault-tolerant consensus protocol has all stakeholders **commit** to a particular block by signing it cryptographically; the protocol is designed such that all honest stakeholders will commit the same block, as long as more than 2/3 of the stakeholders are honest. Thus, we have two **slashing conditions** which leave cryptographic proof that a certain stakeholder is dishonest:

* **Equivocation**, where a stakeholder signs two different blocks with the same block height
* **Invalid block**, where a stakeholder signs an invalid block

In either of these cases, anybody can submit cryptographic evidence \(two conflicting signatures, or a signature on an invalid block\) as a specially-formatted transaction on the blockchain. This **slashing transaction** removes the offending stakeholder, moving all of the staked funds into the reward pool and leaving the offender with nothing. We put slashed funds into the reward pool to incentivize stakeholders --- who already run have the resources to do so --- to audit the behavior of their peers, rather than waiting for random altruistic strangers to do so.

Kicking punishes cryptographically unprovable misbehavior, such as having a very unreliable network, being offline, proposing empty blocks, etc. ~~The exact procedure is rather involved to avoid abuse~~, but the gist of it is that stakeholders holding more than 2/3 of the lents can vote to remove a stakeholder from the list, forcing it to unstake without any reward. This may sound like an easy way for a majority of stakeholders to cartelize and block "competition", but note that such a malicious majority can already do so by simply censoring all staking transactions, so kicking should not pose any additional problem.

### Why our approach?

Celerium's version of bonded proof-of-stake is admittedly a little idiosyncratic, wieh "weird" features like separate staking coins and large minimum stakes. Why do we use proof-of-stake, and why does our version have these other features?

There are many advantages of proof-of-stake over proof-of-work, the other main blockchain consensus mechanism, and the [Ethereum Proof-of-Stake FAQs](https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQs) give a reasonable summary of them. We, however, highlight the two most important reasons:

* PoS allows easy **finality** with Byzantine-fault-tolerant consensus protocols. That is, even in the presence of unreliable and malicious networks, a block that is successfully appended to the blockchain will never be reverted. This completely eliminates a whole class of unpredictable behavior found in "chain-based" consensus protocols like proof of work \(or chain-based PoS as in Peercoin\), such as forks, block reorganizations, and eclipse attacks.
* PoS also allows **stronger incentive-compatibility**, since we can now use the blockchain protocol itself to punish misbehavior by slashing deposits, instead of relying on loss of reputation, opportunity cost, and other harder-to-quantify disincentives. As the Ethereum FAQs put it, "in PoW, we are working directly with the laws of physics. In PoS, we are able to design the protocol in such a way that it has the precise properties that we want - in short, we can optimize the laws of physics in our favor."

There are, however, two main differences between Celerium's proof-of-stake and those of other blockchains.

First, **why does Celerium use a separate currency?** Generally, PoS blockchains use the same coin, like ethers or EOS, as both their main medium of exchange and their staking token. We separate lents from cels because although cels are designed to be currency, lents are essentially equity, and _equity typically does not make a good currency_:

* We want flexible monetary policy for currency \(cels\) to stabilize prices, but unpredictable share dilution \(lents\) disrupts price discovery. Fixed minting schedules are perfect for equity, but terrible for a new currency as they make supply completely inelastic and cause volatile prices.
* Equity demand is ideally constantly adjusting to reflect discounted cash flow \(in our case, future transaction fees\), while demand for currency is driven mostly by the need for a medium of exchange. We want cels to be used and priced purely as money, instead offering lents as a way to "invest in Celerium".

Secondly, **why do we use "tied up" stakes with huge minimum amounts**? This is really because of bandwidth constraints --- new blocks must be broadcast to all validators within a small fraction of the block interval. Thus, we cannot let an unlimited number of validators participate in each round of Byzantine-fault-tolerant consensus. The minimum requirement of 10,000 cels staked per validator places a hard limit of 100 new validators every year, ensuring that growth in the number of validators will not outpace growth in prevailing bandwidth capacity.

Although this is a very simple and obvious mechanism of restricting the validator set, it does not seem to be popular among existing proof-of-stake variants. We think this is most likely because a large minimum staking environment, by "disenfranchising" the vast majority of potential stakeholders, is "politically" unappealing since it sounds superficially like centralization.

Instead, other mechanisms to reduce the number of participants in the core consensus algorithm are used that attempt to

## Auditors: the free press

### Making failure catastrophic

The "free press" in Celerium consists of **auditors**. Auditors are "full nodes" in usual terminology, replicating the entire blockchain --- each block of which is published and cryptographically signed by the stakeholders --- while checking that each transaction is valid with respect to previous ones in the blockchain. They form a random **gossip** network among themselves, similar to that used by Bitcoin full nodes, through which information about new blocks is disseminated. This gossip network reduces load on the stakeholders and makes it difficult for malicious networks to censor the blockchain, since as long as some auditors can connect to the stakeholders and the auditors form a connected graph, new blocks will quickly be visible to every auditor.

The more important role of auditors, though, is to _make consensus failure catastrophic,_ playing a crucial role in keeping the oligarchy of coordinators honest. Auditors utilize their position as relayers of new blocks to continually monitor for evidence that the consensus of the coordinators is corrupt --- for example, invalid blocks or two different blocks at the same height signed by a quorum of coordinators would be proof that the coordinators are no longer trustworthy. These pieces of evidence, known as **consensus nukes**, undeniably prove that at least 1/3 of the stakeholders are actively malicious.

Any auditor that sees a consensus nuke immediately broadcasts it to all auditors it knows in the gossip network, while permanently activating a "kill switch" and refusing to operate normally. Thus, an attempt at forking or appending invalid transactions to the blockchain would figuratively "nuke" the entire network, permanently stopping the system.

### Why do we want this?

This objective seems a little strange. Why would we ever want our network to self-destruct?

The obvious answer is that if we no longer have a 2/3 supermajority of honest stake, the entire system is utterly screwed no matter what. More specifically, the well-known [DLS paper](http://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf) mathematically proves that consensus protocols running in a partially synchronous network model \(that is, network delays are unknown but finite\) cannot possibly tolerate more than 1/3 arbitrary faults. So we have to choose between a model where the network stays up, but malicious stakeholders can corrupt the state arbitrarily \(rewriting history, giving themselves free money --- or shutting down the network\), or one where the only thing a corrupted quorum can do is shut down the network. Clearly, the latter is preferable.

More importantly, consensus nuking changes the incentives of potential attackers by making most attacks unprofitable. Consider a blockchain where consensus-breaking attacks \(like Bitcoin's 51% attack\) allow arbitrary state corruption. A malicious actor with the ability to execute such attacks can extract huge profits --- one easy attack would be to simply sell coins for off-chain currency, break consensus to revert the transaction to the currency trader, and repeat indefinitely. With more complex higher-level applications relying on blockchain data, profit opportunities are even more numerous. Thus, if enough self-serving stakeholders collude, and no reputations and such are on the line, they are greatly incentivized to attack the network and destroy its security guarantees.

If a successful attack can only result in the network stopping all work, only attackers who greatly benefit from destroying the network will participate. Most attackers will not; in fact, since a successful attacker must stake a vast amount of lents to take over more than 1/3 of the stake, destroying the network and thus the value of the investment is highly irrational. Shorting lents appears to be an easy way to profit from a consensus nuke, but fortunately a short position large enough the balance the enormous "long" position needed to own enough stake will be very difficult to set up.

Finally, a shutdown when a successful attack occurs forces Celerium users to manually coordinate an emergency "hard fork" out-of-band to restore the network. This would involve, at the very least, a redistribution of stakes away from the attacking parties, and possibly including protocol improvements to further deter attacks. On the other hand, if the blockchain continues to operate even when stakeholders are corrupting the state, it's concievable that the malicious stakeholder cartel can create a climate of fear or pressure for users to go along with the corrupted chain \(for example, the state corruption might be forced by legal regulation or presented as way of restoring stolen assets\). Consensus nuking ensures that this scenario is impossible.

## Clients: thin yet fully secure

```text
Current thin clients have bad tradeoffs
- Not so thin
- Often trusts too much

Extremely thin by keeping track of only latest header

Fast, safe stakeholder tracking with essentially no "weak subjectivity"z<
```

