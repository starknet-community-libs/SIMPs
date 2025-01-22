**snip**: 28 (temporary number)  
**Title:** Staking v2 proposal  
**Author:** @NatanSW and @ob1337   
Status: Draft  
Type: Standards Track  
Creation date: \*\*  
Github link: 

---

## **Abstract**

By the end of 2025, we plan significant progress on the decentralization of SN. Such progress entails entrusting some core functionalities of SN to SN’s Validators. As a prerequisite for such a significant milestone, we would like to test and verify two things: 

* The incentive structure makes economic sense  
* The Validator set consists of entities capable of running reliable infrastructure. 

Staking v1 was mainly focused on the former, and v2 will be focused on the latter.

Staking v2 should be ready to launch on Starknet mainnet in Q2 2025\. 

In this SNIP, we will describe the staking v2 proposal, its timeline, and its relation to future steps in decentralization in detail.

## **Motivation** 

Starknet continues its steady progression towards decentralization, as outlined in SNIP 18\. We see decentralization as a core requirement for a public blockchain. Currently, Validators are required to run full nodes, but there is no public way to verify this. Without public validation, the performance and reliability of these Validators remain untested. 

Based on our insights from Staking v1 and the current stake distribution, we believe that the economic incentive structure also needs some improvement. To address these challenges, we propose two key enhancements in Staking v2: validator block attestation (which will be included) and an ability to increase Validator commission from its delegators (which will probably be included). 

### Motivation for validator block attestation

Before entrusting SN’s validators with a role that affects SN's consensus (planned by the end of 2025), we ought to ensure that their liveness metrics are sufficient. Demanding liveness from validators will also allow Delegators to compare the liveness and reliability of different Validators, allowing them to make informed staking decisions. 

Validator block attestation will require Validators to attest to randomly selected blocks during each epoch, ensuring they actively track network activity. This mechanism will link Validator rewards to their successful attestation, encouraging consistent network participation. Introducing epochs as part of this design is another stepping stone towards the decentralized protocol version.

This upgrade also provides a valuable opportunity to test how Validators handle protocol changes and allows for improvements in future iterations based on real-world feedback before these changes affect SN stability. 

The need to allow validators to take an active role and show liveness metrics well before they get a formal role in the consensus is what drove this version and what gives high motivation for this feature to be released well before the end of Q2. **Therefore, the other feature in V2, detailed below, has a certain risk of getting postponed to V3.**

### Motivation for commission increase feature

The current commission structure allows the commission to only decrease. While sensible in the short term, this model has a risk of breaking whenever the cost of being a validator goes up. Examples of why the validating cost might increase significantly in the long term include increased usage (which will require the validators to use stronger hardware) or the requirement to produce proofs and not only produce blocks (which is also on our decentralization roadmap). When it becomes unprofitable to be a validator, the validator might exit the protocol entirely, automatically kicking all the users out of participating \- which is extremely disruptive.

However, an unbounded, sudden  0→100 increase in commission is also something we want to avoid \- as we believe DeFi users deserve greater transparency around what commission they will pay in the future, and when their commission might change \- which is why there is more to this feature than simply allowing commission increase.

## **Specification**

### Epochs 

Staking v2 is about adding a “work”, or something that validators should actively do periodically, as a condition to receive rewards. In the future decentralized phase, the staking power of a validator should be an input to the work function (so the more stake you have, the more blocks you need to produce). However, to make things predictable, we can’t have the staking weights map change immediately when someone stakes or unstakes. To handle this, similarly to what is done in many other protocols, we introduce the notion of epochs into the protocol. 

Epochs represent checkpoints where Staker’s staking power is set. This “discrete” view of staking power will be used as an input to the consensus algorithm. Think of each epoch as 30 min to a few hours duration.

Therefore, we introduce a new latency parameter k, and update the staking power and rewards as a result of changes made in epoch i in epoch i+k. In the long term, when validators would produce blocks, k would be greater or equal to 2, as the producer of the first block of epoch j must be known before the last block of epoch j-1. However, in the staking v2 context (where validators are not yet producing blocks), k may be equal one. 

To illustrate with an example, let’s say we have a validator A with 50K STRK staked. During Epoch i, someone delegates to A an additional 10K STRK. during Epoch i+1, someone undelegates from A 30K STRK. Here is what A’s weight and reward are going to look like:

### Block attestation

The primary proposed change in Staking v2 is the introduction of a block attestation mechanism for Validators. This section outlines the technical details of the attestation process. For now, epochs are treated as groups of E blocks (a detailed explanation will follow in a later section).

* Each Validator is assigned a unique identifier per epoch, which equals to Hash(staked\_amount, epoch\_id, validator\_address) % 2^N.    
* During an epoch, Validators have the opportunity to attest to specific blocks. A block qualifies for attestation of its block hash modulo 2^N equal to the identifier  
* Attestation involves submitting an `attest` transaction within a defined block window. If T represents the block number that meets the matching criteria, the attestation window spans from block T+11 to T+W (for instance, where W \= 20). In addition, the attestation must be sent within the corresponding epoch. This means that the last 10 blocks within an epoch cannot be attested.   
* Each Validator is required to perform only one attestation per epoch.

This mechanism ensures Validators actively use full nodes, as they need to track block hashes continuously. Additionally, the attestation reflects their activity and is publicly verifiable. This ensures Validators' reliability is publicly tested, which is crucial before Starknet's security and liveness depend on them.

We note that in this proposal, the work is the same for all  Validators, and only the rewards are proportional to their stake. The reason we opt for this is it makes the implementation more straightforward, so we can focus more time and effort on the later stages. This is essential as testing Validators’ reliability is crucial before handing them core responsibilities. Also, Running a full node is the main cost and effort, which is obligatory for all Validators.

Lastly, the exact parameters of epoch length (E), Modulo power (N), and attestation window (W) will be determined later and added to this proposal, but here are some thoughts about what is bounding them:

* E should be more than 16 to the power of N, but not orders of magnitude more. Notice that with some small probability ((1-1/2^N)^(E \-W) ), validators will not have the opportunity to submit transactions. We want this probability to be small (less than 1%) to keep the rewards variety small. However, too large an E (with respect to N) means that validators with high probability may stop tracking the network until the end of the epoch when a small percentage of the epoch is done  
* E should be large enough to keep the Operator’s cost sensible—even on a day with high gas fees, the yield for an Operator that has staked only 20K STRK over an epoch should be significantly more than the cost of the transaction it would have to submit. E should be small enough so latency on activities that depend on epoch (such as adding or removing stake) will not be too cumbersome.   
* W should be small to ensure liveness. But also \- not too small. We want to incentivize people to react in real-time but also to avoid scenarios where, due to sporadic spikes in the network, the validator failed to submit a transaction in time.

### Rewards 

Rewards are still proportional to stake and calculated using the minting curve. The only change is that they are accumulated only to Validators who performed their attestations in the epoch. Notice that this means we remove the global index of rewards.

In Staking V2, rewards will be “all or nothing” per epoch \- so a validator that submitted a transaction during the epoch which proves he tracked the network will receive all the rewards for the epoch based on his staked amount, while a validator that didn’t do it will get 0 rewards for the epoch’s duration.

After performing the attestation, the rewards that go directly to the validator will accumulate in his account, and the rest will go to this validator’s pool. When a delegator will claim his rewards, instead of looking at the global index we will accumulate all the rewards the delegator has received from all epochs since the last time he claimed rewards. The complexity of this operation will not be O(number of epochs since last claim) but rather O(number of this delegator’s balance changes since last claim), which is assumed to be small enough to fit in one transaction in any real-world-scenario. 

### Latencies 

As we introduce the notion of epochs, the latency from when a user transfers STRK in or out of the staking protocol to the time staking rewards reflect this operation is determined by the remaining time in the current epoch plus the duration of k-1 more epochs. The withdrawal lockup period will still apply after exiting the protocol. (During the withdrawal lockup period, the funds are no longer counted for staking, but are still locked within the staking contract)

\*Although Stake Delegators are subject to a security lockup when withdrawing funds, they can move between Stakers without waiting for the full lockup period, enhancing the delegation market's competitiveness. This pool switching will have an added latency of k epochs when exiting one pool and entering another.

### New protocol parameters

Here is a table summarising the new protocol parameters proposed in this version:

| Economical Parameter | Proposed value |
| :---- | :---- |
| Epoch size (E) | 100-1,000 Blocks (this will change as a function of block time which is 30 seconds now, 2 seconds from 0.14) |
| Modulu power for defining the work (N) | 4-8 (so modulo would be 16-256) |
| The block window range to submit an attestation (W) | 16 \- 30 Block |
| Number of epochs used for latency (k) | 1 |

### Commission increase (if included)

After [exposing](https://community.starknet.io/t/learning-from-staking-v1-a-discussion-on-commission-policy-and-stake-distribution/115163) some routes for allowing commission increase, we plan to opt for something close to option three which was detailed there. Namely:

* Validators will be able to commit to a certain maximum commission M, and the last date that this commitment is relevant for. Until this last date arrives, validators will not be able to increase their commission beyond M, but can freely change their commission in the range \[0,M\]  
* Validators will be able to declare ahead **one** additional commission commitment. For example, a validator that commits to a maximal commission of M=5% until 1.6.2025 can also commit to a maximal commission M’ \= 3% or M’ \= 7% until 1.1.2026. To elaborate more on how it would work:  
  * If M’ \= 7%, then the commission is guaranteed to be at most 5% until 1.6.2025 and to be at most 7% on 1.6.2025-1.1.2026. Except these upper thresholds there is nothing limiting the validator from changing commission.Nothing will happen to the actual commission rate on the 1.6.2025.  
  * If M’ \= 3%, then the commission is guaranteed to be at most 5% until 1.6.2025 and at most 3% from 1.6.2025 to 1.1.2026. Except for these upper thresholds, nothing limits the validator from changing the commission. On 1.6.2025, if the current commission is higher than 3%, it will be forced to drop to 3%. Defacto, this will be forced whenever the validator does some work (so relatively quickly on staking v2, but might take a bit longer time in future staking versions).   
* Once given, the validator cannot change the commitments. The validator also can’t include more than one commitment that refers to a future date (so in the example above, no further commitments can be given until 1.1.1.2026)  
* Commitments may not refer to more than one year from the date they are given. The purpose here is to be future-compatible with other features around commission (such as imposing minimal commission for large stakers). When the protocol stabilizes, the period the commitment refer to will be extended.   
* One edge case that the description above doesn’t cover is if/how to prevent validators to go 0→100 commission the day after v2 is released, when no commitments are yet done. To counter that, we consider to prevent any commission increase in the two first months after v2 goes live, giving the users ample of time to switch validators according to the relevant commitments made by the different validators. 

We encourage validators to impose limitations on their commission to give maximal transparency to the users. We will work with staking platforms and dashboards to highlight validators without commitment, or validators where the current commitment will be expired within the next month with no followup commitment, as a "dangerous" validator where commission could suddenly change unexpectedly. 

## **Backward compatibility** 

**Rewards**  
All rewards accumulated before the upgrade will be accessible and will be acocunted the first time validator or delegator performs an operation on the contract (claim, deposit or withdraw STRK for delegators, perform work for validators). As usual, these rewards will be transferred out only on claim. 

Accumulating rewards after the upgrade will happen only if Validator will attest to blocks  
Getting rid of the Index structure on the Staking contract all together.  
Cost of reward-claim by delegator will now depend on the number of times this delgator changed his balance since last claim

**Epochs upon entering and exiting.**

* Users that enter the protocol on epoch i will start get rewards on epoch i+k  
* Users that intent  exit the protocol on epoch i will still get rewards on epoch i till i+k \- 1

**Economical feature**

* Validators will be able to increase commission

**Actual potential ABI and events \- we will update here when will know more.**

## **Security Considerations**

### New security considerations 

1. **Operational address is now used** \- the operational address Valdiators provided upon registration will now be used for submitting transactions to the network. Hacking the operational address can now lead to lose of yield for the validator and its delegators  
2. Preventing fast switching between validators \- to prevent delegator behaviours where delegators that see their validator fails to perform work to too-quickly move a validator and enjoy the work it has performed. To prevent this, switching declared on epoch i will take effect in epoch i+1. 

The security considerations are the same as they have been on Staking V1 ([SNIP 18](https://community.starknet.io/t/snip-18-staking-s-first-stage-on-starknet/114334)). They are restated below for the ske of self-containment. 

The staking protocol's design includes several key security measures to ensure its integrity and user safety.

**Modular architecture** 

The proposed protocol will be implemented using a modular architecture, where different functionalities are separated into distinct contracts, such as staking, delegation, rewards supply, and more. This approach offers several advantages:

* **Simplifies Logic:** Each contract has a clear and specific role, making the code easier to test, audit, and understand.  
* **Enhances Access Control:** By defining specific permissions for each contract, the risk of unauthorized access or misuse is reduced.  
* **Allows Targeted Upgrades and Changes:** Updating only the relevant contract minimizes the potential for introducing new vulnerabilities.

### **Permissions and Key Management**  

The Staking protocol allows users to define different addresses for different functionalities. Thus, cold addresses with minimal activity can control important functionalities, reducing exposure to threats and enhancing user security.

Stakers register with three addresses:

* **Staking and Unstaking Address**: This address has permission to Stake, add Stake and unstake. This means that it handles large amounts of STRK tokens and is only needed when entering or exiting the protocol. Thus, it can be kept by a cold wallet (minimal activity) to maximize security.  
* **Rewards Address**: This is the address where rewards will be sent to. It can also be maintained with minimal activity, i.e., as a cold wallet.  
* **Operational Address**: This address represents the Staker for operational purposes, for example, block attestations, and in the future block proposing etc. Note that this address does not handle large amounts of funds. As it will be frequently used, it should be categorized as a hot address.

Stake Delegators register with two addresses:

* **Entering/Exiting Address**: This address has permission to Delegate Stake, add Stake and unstake. This means that it handles large amounts of STRK tokens and is only needed when entering or exiting the protocol. Thus, it can be kept by a cold wallet (minimal activity) to maximize security.  
* **Rewards Address**: This is the address where rewards will be sent to. It can also be maintained with minimal activity, i.e., as a cold wallet.

**Permissions:** The protocol uses a hierarchical approach between user roles. Colder addresses, which control larger funds, can replace more active ones if compromised. Specifically, Staker and Stake Delegator addresses can replace the rewards address.

## **Security lockup period**

When exiting, users face a 21-day lockup, during which no rewards are earned, and funds cannot be withdrawn. This disincentivises sudden large withdrawals that could destabilize the network. Future versions will tie the exit process to epochs, maintaining the lockup period for a secure exit mechanism.

# **Timeline**

The development work is well underway, and the first stage of stakes is planned to go live as early as Q2 2024\. You can follow the development on our open-source [repo](https://github.com/starkware-libs/starknet-staking).

Future stages will include Stakers gradually assigned responsibilities, eventually validating and sequencing blocks. This gradual transition ensures stability and allows for thorough testing and improvements. For more details on the decentralized protocol, check out @Ilia’s [series](https://community.starknet.io/t/starknet-decentralized-protocol-i-introduction/2671) of blog posts.

More details and proposals on subsequent stages will be introduced, as we progress. The next step is already in the research and planning stages, set to launch a few months after the second stage. Our objective is to roll out these updates smoothly, with ongoing improvements guided by community feedback.

# **Feedback**

We want you to share your feedback and ask questions. You can comment on the community blog post and follow our development on our open-source [repo](https://github.com/starkware-libs/starknet-staking).

## **Copyright**

This SNIP will be released under an MIT license.