---
layout: post
title:  "paxos"
date:   2018-6-4 16:47:53
categories: paxos
---
### Introduction(介绍)

```c
The Paxos algorithm for implementing a fault-tolerant distributed system has been regarded as difficult to understand, 
perhaps because the original presentation was Greek to many readers [5]. In fact, it is among the simplest
and most obvious of distributed algorithms. At its heart is a consensus
algorithm—the “synod” algorithm of [5]. The next section shows that this
consensus algorithm follows almost unavoidably from the properties we want
it to satisfy. The last section explains the complete Paxos algorithm, which
is obtained by the straightforward application of consensus to the state machine
approach for building a distributed system—an approach that should
be well-known, since it is the subject of what is probably the most often-cited
article on the theory of distributed systems [4].
```

Paxos算法实现了一个容错的分布式系统，它往往被认为是难以理解的，可能因为最早的表述对于大多数读者来说是读希腊故事[5]。

事实上，在所有分布式算法中，Paxos算法是最简单的，也是最显而易见的。

其核心就是一个一致性算法-the “synod” algorithm of [5].

从下面的章节可以看出，这种一致性算法不可避免地满足了我们希望它满足的性质。

最后一节解释了完整的Paxos算法，可以通过构造分布式系统的一个众所周知的方法来得到，即，从一致性到状态机的直接应用。

他也是在分布式理论论文中经常引用的一个主题。

### The Consensus Algorithm(一致性算法)

#### The Problem(问题)

```c
Assume a collection of processes that can propose values. A consensus algorithm
ensures that a single one among the proposed values is chosen. If
no value is proposed, then no value should be chosen. If a value has been
chosen, then processes should be able to learn the chosen value. The safety
requirements for consensus are:
• Only a value that has been proposed may be chosen,
• Only a single value is chosen, and
• A process never learns that a value has been chosen unless it actually
has been.
```

假定一组进程可以提议一些values。有一种一致性算法，它确保在所有被提议的values中，选择(chosen)其中一个。

如果没有value被提议，那么就没有value被选择(chosen)。如果有一个value被选择，那么进程应该可以学习此value。

一致性的安全需求如下：

. 只有被提议(proposed)的value才能被选择(chosen)

. 只有一个value被选择

. 一个进程只能学习确定被选择(chosen)的value

```c
We won’t try to specify precise liveness requirements. However, the goal is
to ensure that some proposed value is eventually chosen and, if a value has
been chosen, then a process can eventually learn the value.
```

我们并不会尝试指出精确的活性要求。然而，目的是确保一些被提议(proposed)的value最终能被选择(chosen)，

如果一个value被选择(chosen)，那么一个进程最终能学习到此value。

```c
We let the three roles in the consensus algorithm be performed by three
classes of agents: proposers, acceptors, and learners. In an implementation,
a single process may act as more than one agent, but the mapping from
agents to processes does not concern us here.
```

我们让三类代理，如proposers, acceptors, and learners来扮演分布式算法中的三种角色。

对于某一种具体实现，一个单独的进程可能扮演多种角色。然而，我们这里并不会关心这种角色的具体分工。

```c
Assume that agents can communicate with one another by sending messages.
We use the customary asynchronous, non-Byzantine model, in which:

. Agents operate at arbitrary speed, may fail by stopping, and may
restart. Since all agents may fail after a value is chosen and then
restart, a solution is impossible unless some information can be remembered
by an agent that has failed and restarted.
. Messages can take arbitrarily long to be delivered, can be duplicated,
and can be lost, but they are not corrupted.
```

假如这些角色代理之间可以通过发送消息来通信。

我们采用通常意义的异步，没有拜占庭问题的通信模型，为此：

. 代理可以任意速度运行，可能由于停止服务而故障，可能会重启。由于所有的代理服务可能在一个value被chosen后故障，然后又重启成功,
  所以一个可能的解决方案是代理能对关键信息进行持久化保存。
  
. 消息分发的时延是任意长的，允许重复，允许丢失，但是消息内容不能发生错误(译者注：非拜占庭问题)。


#### Choosing a Value(选择一个值)
```c
The easiest way to choose a value is to have a single acceptor agent. A proposer
sends a proposal to the acceptor, who chooses the first proposed value
that it receives. Although simple, this solution is unsatisfactory because the
failure of the acceptor makes any further progress impossible.
```

选择一个value最简单的方法就是只有一个acceptor。某proposer发送一个proposal给acceptor，然后acceptor选择它第一次收到的proposed value。

尽管简单，然而不能满足我们的要求。因为acceptor agent故障会导致后续的工作没法进行。

```c
So, let’s try another way of choosing a value. Instead of a single acceptor,
let’s use multiple acceptor agents. A proposer sends a proposed value to a
set of acceptors. An acceptor may accept the proposed value. The value is
chosen when a large enough set of acceptors have accepted it. How large is
large enough? To ensure that only a single value is chosen, we can let a large
enough set consist of any majority of the agents. Because any two majorities
have at least one acceptor in common, this works if an acceptor can accept
at most one value. (There is an obvious generalization of a majority that
has been observed in numerous papers, apparently starting with [3].)
```

因此，我们还得尝试别的方法。和单个acceptor比较而言，我们采用多个acceptor。某proposer给一组acceptor发送proposed value。

某些acceptor可能会接受proposed value。如果有足够大的多数派acceptor集合接受了此proposed value，我们认为此value被选择。

多大算是足够大呢？为了确保只有一个value被选择(chosen), 我们让一个足够大的集合由任意多数派组成。

因为任意两个多数派至少会有一个共同的acceptor，所以只要一个acceptor最多只接受一个value，那么这个办法是可行的。

（这种显而易见的多数派，在许多论文中看到过，最明显的起始于[3])

```c
In the absence of failure or message loss, we want a value to be chosen
even if only one value is proposed by a single proposer. This suggests the
requirement:

P1. An acceptor must accept the first proposal that it receives.
```

在没有故障或者消息丢失的情况下，及时只有一个proposer提议了一个value，我们要求选择(chosen)此value。

这提出了要求：

P1. acceptor必须接受他收到的第一个提案。

```c
But this requirement raises a problem. Several values could be proposed by
different proposers at about the same time, leading to a situation in which
every acceptor has accepted a value, but no single value is accepted by a
majority of them. Even with just two proposed values, if each is accepted by
about half the acceptors, failure of a single acceptor could make it impossible
to learn which of the values was chosen.
```

但是这个要求引发了一个问题。不同的proposer可能会同时提议value，这可能会导致一种情况：

尽管每一个acceptor接受了一个value，但是没有一个value被大多数acceptor接受。

甚至在只有两个value被提议的场景下，假如刚好每一半acceptor分别接受了这两个value，

某个acceptor发生故障会导致无法确定哪个value被选择。(译者注：这是要说acceptor的个数是奇数的情况下吗？)

```c
P1 and the requirement that a value is chosen only when it is accepted
by a majority of acceptors imply that an acceptor must be allowed to accept
more than one proposal. We keep track of the different proposals that an
acceptor may accept by assigning a (natural) number to each proposal, so a
proposal consists of a proposal number and a value. To prevent confusion,
we require that different proposals have different numbers. How this is 
achieved depends on the implementation, so for now we just assume it. A
value is chosen when a single proposal with that value has been accepted by
a majority of the acceptors. In that case, we say that the proposal (as well
as its value) has been chosen.
```

P1和多数派接受才能选择一个value的要求，意味着acceptor必须能接受多个提案。

我们通过给每一个提案编号来跟踪不同的提案，因此一个提案由编号和value组成。

为了防止混淆，我们要求提案不同，编号不同。如何实现此目标依赖具体实现，目前为止，我们仅仅是假设而已。

当多数派acceptor接受了一个提案，提案携带的value被选择。此时，我们说提案被选择。

```c
We can allow multiple proposals to be chosen, but we must guarantee
that all chosen proposals have the same value. By induction on the proposal
number, it suffices to guarantee:

P2. If a proposal with value v is chosen, then every higher-numbered proposal
that is chosen has value v.
```

我们允许选择多个提案，但是我们必须确保这些被选择的提案有相同的vlaue。(译者注：同一个acceptor可以接受提案。但是他们的个value必须相同, 为了满足“安全属性":只有一个值被选择)

通过对提案编号进行归纳，就足以保证：

P2. 如果拥有value v的提案被选择，那么每一个被选择的更大编号的提案也拥有value v。

```c
  Since numbers are totally ordered, condition P2 guarantees the crucial safety
property that only a single value is chosen.
```

  由于编号是全序的，所以条件P2确保了关键安全属性：只有一个value被选择

```c  
  To be chosen, a proposal must be accepted by at least one acceptor. So,
we can satisfy P2 by satisfying:

P2a. If a proposal with value v is chosen, then every higher-numbered proposal
accepted by any acceptor has value v.
```

  一个提案要想被选择，那么至少被一个acceptor接受。因此，我们可以通过满足如下条件来满足P2:
 
P2a. 如果拥有value v的提案被选择，那么任意acceptor接受的每一个更大编号的提案拥有value v。(译者注：约束acceptor)

```c 
We still maintain P1 to ensure that some proposal is chosen. Because communication
is asynchronous, a proposal could be chosen with some particular
acceptor c never having received any proposal. Suppose a new proposer
“wakes up” and issues a higher-numbered proposal with a different value.
P1 requires c to accept this proposal, violating P2a. Maintaining both P1 and P2a
requires strengthening P2a to:

P2b. If a proposal with value v is chosen, then every higher-numbered proposal
issued by any proposer has value v.
```

为了确保提案能被选择，我们任然需要维护P1. 因为通信是异步的，某个提案可能被一个从来没有接收过任何提案的acceptor c所选择。

假设一个新醒来的proposer发送了一个带有不同value的更大编号的提案。

P1 要求c接收此提案，违反了P2a. 同时维护P1和P2a需要加强P2a到：

P2b. 如果拥有value v的提案被选择，那么任意proposer发送的每一个更大编号的提案拥有value v。(译者注：约束proposer)

```c
Since a proposal must be issued by a proposer before it can be accepted by
an acceptor, P2b implies P2a, which in turn implies P2.
```

由于一个提案被一个acceptor接受之前，它必须由proposer发送，那么由P2b蕴含着P2a，从蕴含着P2.

```c
To discover how to satisfy P2b, let’s consider how we would prove that
it holds. We would assume that some proposal with number m and value
v is chosen and show that any proposal issued with number n > m also
has value v. We would make the proof easier by using induction on n,
so we can prove that proposal number n has value v under the additional
assumption that every proposal issued with a number in m . .(n − 1) has
value v, where i . . j denotes the set of numbers from i through j. For the
proposal numbered m to be chosen, there must be some set C consisting of a
majority of acceptors such that every acceptor in C accepted it. Combining
this with the induction assumption, the hypothesis that m is chosen implies:

Every acceptor in C has accepted a proposal with number in
m . .(n − 1), and every proposal with number in m . .(n − 1)
accepted by any acceptor has value v.
```

为了发现如何满足P2b, 我们可以考虑如何来证明P2b是成立的。

我们可以假设编号为m，value是v的提案被选择，能够得出任何被发送的编号满足条件n>m的提案也拥有value v。

我们可以对编号n进行归纳，做一个简单的证明。

因此我们要做的是，假设每一个被发送的编号是m..(n-1)的提案拥有value v(其中i..j表示从i到j的编号集合)，能够证明编号为n的提案拥有value v。（此条是归纳假设)

对于编号为m的提案被选择，那么必然存在由多数派acceptor组成的集合C，以至于集合C中的每一个acceptor接受了此提案。

联合词条和归纳假设，假设m被选择，蕴含着：

集合C中的每一个acceptor已经接受了编号m..(n-1)的提案，并且每一个被任意acceptor接受的拥有编号m..(n-1)的提案拥有value v。

```c
Since any set S consisting of a majority of acceptors contains at least one
member of C , we can conclude that a proposal numbered n has value v by
ensuring that the following invariant is maintained:

P2c. For any v and n, if a proposal with value v and number n is issued,
then there is a set S consisting of a majority of acceptors such that
either (a) no acceptor in S has accepted any proposal numbered less
than n, or (b) v is the value of the highest-numbered proposal among
all proposals numbered less than n accepted by the acceptors in S.

We can therefore satisfy P2b by maintaining the invariance of P2c.
```

由于任何由多数派组成的集合S至少包含了一个集合C中的成员，我们可以通过确保维护下面的不变性得出编号为n的提案拥有value v：

P2c. 对于任意v和n，如果一个拥有value v和编号n的提案被发送，那么存在一个有多数派acceptor组成的集合S，以至于：

(a) 没有集合S中的acceptor曾经接受过任意编号小于n的提案。或者，(b) 由集合S所接受的所有编号小于n提案中，编号最大的提案的value是v。

因此我们可以通过维护P2c的不变性来满足条件P2b.

```c
  To maintain the invariance of P2c, a proposer that wants to issue a proposal
numbered n must learn the highest-numbered proposal with number
less than n, if any, that has been or will be accepted by each acceptor in
some majority of acceptors. Learning about proposals already accepted is
easy enough; predicting future acceptances is hard. Instead of trying to predict
the future, the proposer controls it by extracting a promise that there
won’t be any such acceptances. In other words, the proposer requests that
the acceptors not accept any more proposals numbered less than n. This
leads to the following algorithm for issuing proposals.

1. A proposer chooses a new proposal number n and sends a request to
each member of some set of acceptors, asking it to respond with:
  (a) A promise never again to accept a proposal numbered less than n, and
  (b) The proposal with the highest number less than n that it has
accepted, if any.

I will call such a request a prepare request with number n.

2. If the proposer receives the requested responses from a majority of
the acceptors, then it can issue a proposal with number n and value
v, where v is the value of the highest-numbered proposal among the
responses, or is any value selected by the proposer if the responders
reported no proposals.
```

  为了维护P2c的不变性，一个proposer要想发送一个编号为n的提案，那么对于任意已经或者将要被多数派接受的提案，他必须学习提案编号比n小的最大编号。

学习已经接受的提案是很简单的，预测未来接受的时候困难的。我们并不是要尝试预测未来，proposer通过获得将来不会再有这样接受的承诺，去控制“学习提案编号比n小的最大编号”。

换句话说，proposer请求acceptors不在接受任意编号小于n的提案。这会得出如下的算法，用于发送提案：

1. proposer向acceptor集合中的每一位成员发送编号为n的提案请求，期望得到的应答如下：
  
  (a) 承诺不再会接受编号小于n的提案，或者，
  
  (b) 它(acceptor)接受的所有提案的编号小于n。

我把这样的请求称做编号为n的prepare request。

2. 如果proposer收到了多数派对prepare request的应答，那么它可以发送一个编号是n、value为v的提案，其中v就是所有响应的提案中编号最大的提案的value

或者是如果应答人没有报告提案，proposer可以选择任意value。

```c
  A proposer issues a proposal by sending, to some set of acceptors, a request
that the proposal be accepted. (This need not be the same set of acceptors
that responded to the initial requests.) Let’s call this an accept request.
```

  proposer发送提案请求给一组acceptor。(这组acceptor和接收prepare request的acceptors可以不是一组)
  
我们把这个请求叫做accept request。

```c
  This describes a proposer’s algorithm. What about an acceptor? It can
receive two kinds of requests from proposers: prepare requests and accept
requests. An acceptor can ignore any request without compromising safety.
So, we need to say only when it is allowed to respond to a request. It can
always respond to a prepare request. It can respond to an accept request,
accepting the proposal, if it has not promised not to. In other words:

P1a. An acceptor can accept a proposal numbered n if it has not responded
to a prepare request having a number greater than n.

Observe that P1a subsumes P1.
```

  上面描述了proposer算法。acceptor的算法是什么呢？acceptor可以从proposer接受到两类请求： prepare requests and accept requests。

acceptor可以忽略任意请求，不会破坏安全性。因此，我们应该说只有当acceptor被允许应答proposer的请求时，acceptor才会应答。

acceptor可以总是响应prepare request。它也可以接受一个提案，去响应accept request，如果他没有承诺不这样做的话。换句话说：

P1a. 如果acceptor曾经没有对编号大于n的prepare request做出响应，那么它可以接受一个编号为n的提案。

通过观察可以发现，P1a预示着P1。

```c
  We now have a complete algorithm for choosing a value that satisfies the
required safety properties—assuming unique proposal numbers. The final
algorithm is obtained by making one small optimization.
```

  现在我们有完整的算法来选择满足安全属性的value。最终的算法需要做少许优化：

```c  
  Suppose an acceptor receives a prepare request numbered n, but it has
already responded to a prepare request numbered greater than n, thereby
promising not to accept any new proposal numbered n. There is then no
reason for the acceptor to respond to the new prepare request, since it will
not accept the proposal numbered n that the proposer wants to issue. So
we have the acceptor ignore such a prepare request. We also have it ignore
a prepare request for a proposal it has already accepted.
```

  假定一个acceptor接收了编号为n的prepare request，但是他已经对一个编号大于n的prepare request做出了应答，

所以它得承诺不能再接受任意编号为n的新的提案。那样，acceptor再没有理由去响应新的prepare request。

至此，他将不会再接受proposer想要发送，但是编号大于n的提案。

因此，我们将会使acceptor忽略这样的prepare request。我们当然也会让acceptor忽略已经接受的prepare request。

```c
  With this optimization, an acceptor needs to remember only the highestnumbered
proposal that it has ever accepted and the number of the highestnumbered
prepare request to which it has responded. Because P2c must
be kept invariant regardless of failures, an acceptor must remember this
information even if it fails and then restarts. Note that the proposer can
always abandon a proposal and forget all about it—as long as it never tries
to issue another proposal with the same number.
```

有了这样的优化，acceptor只需要记录他曾经接受的编号最大的提案和他曾经响应的编号最大的prepare request的number。

因为不管什么故障P2c必须要保持不变性，acceptor必须记录这些信息，及时它故障又重启。

需要注意的是，proposer可以总是丢弃某个提案，并且只要他不在发送相同编号的提案，他可以忘记和此提案相关的所有信息。

```c
  Putting the actions of the proposer and acceptor together, we see that
the algorithm operates in the following two phases.

Phase 1. (a) A proposer selects a proposal number n and sends a prepare
request with number n to a majority of acceptors.
(b) If an acceptor receives a prepare request with number n greater
than that of any prepare request to which it has already responded,
then it responds to the request with a promise not to accept any more
proposals numbered less than n and with the highest-numbered proposal
(if any) that it has accepted.

Phase 2. (a) If the proposer receives a response to its prepare requests
(numbered n) from a majority of acceptors, then it sends an accept
request to each of those acceptors for a proposal numbered n with a
value v, where v is the value of the highest-numbered proposal among
the responses, or is any value if the responses reported no proposals.
(b) If an acceptor receives an accept request for a proposal numbered
n, it accepts the proposal unless it has already responded to a prepare
request having a number greater than n.
```

  把proposer和acceptor的行为集中在一起，我们可以发现算法的执行分两个阶段：
  
Phase 1. (a) proposer选择一个提案编号n，发送编号为n的prepare request给多数派acceptor。
         
		 (b) 如果acceptor收到了一个编号为n的prepare request，但是他曾经响应的所有prepare request的编号小于n，

		那么，他会承诺不会接受任何编号小于n的提案，并且应答他曾经接受的所有提案中最大的number。

Phase 2. (a) 如果proposer收到了来自多数派acceptor对他prepare requests(编号n)的应答，那么，
       
	      他会发送一个编号为n，value是v的提案给acceptors。如果应答消息中有value，那么v选择编号最大的应答的value；
		  
		  如果应答中没有value，那么v可以是任意value。
		  
		 (b) 如果acceptor收到了一个编号为n的提案的accept request，那么它会接受此提案，除非是他曾经竞答过一个编号大于n的prepare request。

 ```c
  A proposer can make multiple proposals, so long as it follows the algorithm
for each one. It can abandon a proposal in the middle of the protocol at any
time. (Correctness is maintained, even though requests and/or responses
for the proposal may arrive at their destinations long after the proposal
was abandoned.) It is probably a good idea to abandon a proposal if some
proposer has begun trying to issue a higher-numbered one. Therefore, if an
acceptor ignores a prepare or accept request because it has already received
a prepare request with a higher number, then it should probably inform
the proposer, who should then abandon its proposal. This is a performance
optimization that does not affect correctness.
```

  只要遵守算法规则，一个proposer可以产生很多提案。他可以在协议过程的任何时间点丢弃某个提案。(尽管当提案被丢弃以后，此提案的请求和/或者响应可能才到达目的地，
  
然而正确性是能够保证的。)如果有proposer尝试发送更大编号的提案，那么选择丢弃一个提案很可能是很好的想法。
  
因此，假如一个acceptor由于接受了更大编号的prepare request，而忽略一个prepare or accept request，那么它应该通知想丢弃此提案的propser。
  
这是一个性能优化，并不会影响正确性。

 
2.3 Learning a Chosen Value(学习被选择的值)

```c
  To learn that a value has been chosen, a learner must find out that a proposal
has been accepted by a majority of acceptors. The obvious algorithm
is to have each acceptor, whenever it accepts a proposal, respond to all
learners, sending them the proposal. This allows learners to find out about
a chosen value as soon as possible, but it requires each acceptor to respond
to each learner—a number of responses equal to the product of the number
of acceptors and the number of learners.  
```

  为了学习一个选定的值，learner必须找出被多数派接受的提案。显而易见的算法是，acceptor无论在什么时候接受了一个提案，都应该发送给learner。

这允许learners尽可能的找出被选择的value，这要求每一个acceptor响应每一个learner。响应的数量等于acceptors个数和learners个数的乘积。

```c
  The assumption of non-Byzantine failures makes it easy for one learner
to find out from another learner that a value has been accepted. We can
have the acceptors respond with their acceptances to a distinguished learner,
which in turn informs the other learners when a value has been chosen. This
approach requires an extra round for all the learners to discover the chosen
value. It is also less reliable, since the distinguished learner could fail. But
it requires a number of responses equal only to the sum of the number of
acceptors and the number of learners.
```

  非拜占庭故障的假设，使得某个learner从另一个learner找出被接受的value变得很简单。
  
我们可以让所有acceptors把他们接受的提案通知给不同的learner，每一个learner在通知其他learner。

要想让所有learner发现被选择的value，这种方法额外需要一轮。这也不太可靠，因为learner可能会故障。

但是，响应的数量等于acceptors个数和learners个数的和。

```c
  More generally, the acceptors could respond with their acceptances to 
some set of distinguished learners, each of which can then inform all the
learners when a value has been chosen. Using a larger set of distinguished
learners provides greater reliability at the cost of greater communication
complexity.
```

  更通用的，acceptors可以将接收的提案发送给不同learners集合，每一个learner再将被选择的value通知给其他所有learner。
  
  如果不同learner的集合越大，那么相应的通信复杂度月更大，带来的服务更可靠。

```c
  Because of message loss, a value could be chosen with no learner ever finding out. 
The learner could ask the acceptors what proposals they have
accepted, but failure of an acceptor could make it impossible to know whether
or not a majority had accepted a particular proposal. In that case, learners
will find out what value is chosen only when a new proposal is chosen. If
a learner needs to know whether a value has been chosen, it can have a
proposer issue a proposal, using the algorithm described above.
```

  由于消息会丢失，被选择的value，没有任何learner知道。learner可以询问acceptors所接受的提案，
  
但是由于acceptor故障，使得他不可能知道是否某个提案是被多数派接受的。在那种情况下，learner只需要在新的提案被选择的时候，去找出被选择的value。

如果learner需要知道是否一个value被选择，他可以让proposer重新发送一个提案。

2.4 Progress（进度）

```c
  It’s easy to construct a scenario in which two proposers each keep issuing
a sequence of proposals with increasing numbers, none of which are ever
chosen. Proposer p completes phase 1 for a proposal number n1. Another
proposer q then completes phase 1 for a proposal number n2 > n1. Proposer
p’s phase 2 accept requests for a proposal numbered n1 are ignored because
the acceptors have all promised not to accept any new proposal numbered
less than n2. So, proposer p then begins and completes phase 1 for a new
proposal number n3 > n2, causing the second phase 2 accept requests of
proposer q to be ignored. And so on.
```

  可以很容易的构造一个场景，两个proposers发送序号递增的提案，但是没有一个提案会被选择。
  
Proposer p对提案号n1完成了phase 1。另一个proposer q对提案号n2 > n1完成了phase1。

Proposer P在phase 2的编号为n1的提案的accept request被忽略了，因为acceptors已经做出了

不再接受编号小于n2的任何新的提案。因此，proposer p用新的编号n3>n2来完成phase 1，

这会导致proposer q在phase 2的 accept requests被忽略。然后，重复此过程。

```c
  To guarantee progress, a distinguished proposer must be selected as the
only one to try issuing proposals. If the distinguished proposer can communicate
successfully with a majority of acceptors, and if it uses a proposal
with number greater than any already used, then it will succeed in issuing a
proposal that is accepted. By abandoning a proposal and trying again if it
learns about some request with a higher proposal number, the distinguished
proposer will eventually choose a high enough proposal number.
```

  为了确保进度，必须选择一个知名的proposer作为发送提案的唯一一个人。如果知名proposer可以和多数派acceptors正常通信，
  
并且如果他用了一个提案，其编号大于任何已经用的，那么发送一个想要被接受的提案会成功。

如果得知一些请求带有更大的提案号，可以放弃此提案，并且再次尝试，知名proposer最终会找到足够大的提案编号。

```c  
  If enough of the system (proposer, acceptors, and communication network)
is working properly, liveness can therefore be achieved by electing a
single distinguished proposer. The famous result of Fischer, Lynch, and Patterson
[1] implies that a reliable algorithm for electing a proposer must use
either randomness or real time—for example, by using timeouts. However,
safety is ensured regardless of the success or failure of the election.
```

如果系统中大多数角色(proposer, acceptors, and communication network)运行的很好，

可以通过选举一个知名proposer完成活性。Fischer, Lynch, and Patterson的知名结论，

预示着，选举一个proposer的可靠性算法要么使用随机性，要么使用实时时间。例如，可以用timeouts。

然而，无论选举时成功还是失败，安全性是可以保证的。

2.5 The Implementation(实现)

```c
  The Paxos algorithm [5] assumes a network of processes. In its consensus
algorithm, each process plays the role of proposer, acceptor, and learner.
The algorithm chooses a leader, which plays the roles of the distinguished
proposer and the distinguished learner. The Paxos consensus algorithm is
precisely the one described above, where requests and responses are sent as
ordinary messages. (Response messages are tagged with the corresponding
proposal number to prevent confusion.) Stable storage, preserved during
failures, is used to maintain the information that the acceptor must remember.
An acceptor records its intended response in stable storage before
actually sending the response.
```

  Paxos算法假设了网络中的一组进程。在它的一致性算法中，每一个进程扮演了proposer, acceptor, and learner角色。

算法会选举一个leader，扮演了知名proposer和知名learner的角色。Paxos一致性算法就是前面描述的，请求和响应是以普通的消息来发送的。

(防止迷惑，响应消息带了相应的提案号)。在故障阶段，稳定存储用以维护acceptor必须记录的信息。

在发送响应前，An acceptor需要将响应消息记录在存储中。

```c
  All that remains is to describe the mechanism for guaranteeing that no
two proposals are ever issued with the same number. Different proposers
choose their numbers from disjoint sets of numbers, so two different proposers
never issue a proposal with the same number. Each proposer remembers
(in stable storage) the highest-numbered proposal it has tried to issue,
and begins phase 1 with a higher proposal number than any it has already
used.
```

  接下来会描述一种确保没有两个提案拥有相同编号的机制。不同的proposer从不相交的编号集合中选择自己的编号，

因此两个不同的proposer永远不会发送编号相同的提案。每一个propser需要在存储记录想要发送的编号最大的提案，

开始phase 1，选取的提案号要大于曾经使用过的提案号中的最大值。

```c
  To guarantee that all servers execute the same sequence of state machine
commands, we implement a sequence of separate instances of the Paxos
consensus algorithm, the value chosen by the ith instance being the ith state
machine command in the sequence. Each server plays all the roles (proposer,
acceptor, and learner) in each instance of the algorithm. For now, I assume
that the set of servers is fixed, so all instances of the consensus algorithm
use the same sets of agents.
```

  为了确保所有服务执行相同的状态机命令序列，我们实现了一系列单独的Paxos一致性算法实例，
  
由序列中第i个实例选择的value将是第i个状态机命令。在算法的实例中，每一个服务扮演了所有的角色(proposer,acceptor, and learner)。

到目前为止，我假定了服务器的集合是固定的，因此，一致性算法的所有实例使用了相同的代理集合。

```c
  In normal operation, a single server is elected to be the leader, which
acts as the distinguished proposer (the only one that tries to issue proposals)
in all instances of the consensus algorithm. Clients send commands to the
leader, who decides where in the sequence each command should appear.
If the leader decides that a certain client command should be the 135th
command, it tries to have that command chosen as the value of the 135th
instance of the consensus algorithm. It will usually succeed. It might fail
because of failures, or because another server also believes itself to be the
leader and has a different idea of what the 135th command should be. But
the consensus algorithm ensures that at most one command can be chosen
as the 135th one.
```

  在正常操作中，某个server被选为leader。leader在一致性算法的所有实例中扮演了知名的proposer(唯一可以发送提案的那个人)。
  
客户端把命令发给leader，由leader决定此命令应该处于命令序列中的哪个位置。

如果leader认为某个客户端命令应该是第135个命令，那么leader会试图让一致性算法中第135个实例被选择，此客户端命令作为实例的value被选择。

大多数情况会成功的。不过，有时会因为故障，或者是由于别的服务认为自己是leader，导致了到底谁是第135个命令有了不同的答案。

然而一致性算法可以确保只有一个命令被选择为第135个命令。

```c
  Key to the efficiency of this approach is that, in the Paxos consensus
algorithm, the value to be proposed is not chosen until phase 2. Recall that,
after completing phase 1 of the proposer’s algorithm, either the value to be
proposed is determined or else the proposer is free to propose any value.
```

  此方法的性能关键是，直到phase 2，被提议的value才能被选择。回忆一下，
  
当proposer的算法phase 1完成后，要么要被提议的value已经被选择，要么proposer可以随意确定任意value。

```c  
  I will now describe how the Paxos state machine implementation works
during normal operation. Later, I will discuss what can go wrong. I consider
what happens when the previous leader has just failed and a new leader has
been selected. (System startup is a special case in which no commands have
yet been proposed.)
```

  现在我会描述Paxos状态机在正常操作期间的实现原理。随后，我会讨论可能会发生的错误。
  
当之前的leader已经故障并且新的leader被选举，我会考虑期间发生什么。

(系统启动阶段是一个特殊情况，期间没有任何命令被提议)

```c
  The new leader, being a learner in all instances of the consensus algorithm,
should know most of the commands that have already been chosen.
Suppose it knows commands 1–134, 138, and 139—that is, the values chosen
in instances 1–134, 138, and 139 of the consensus algorithm. (We will
see later how such a gap in the command sequence could arise.) It then
executes phase 1 of instances 135–137 and of all instances greater than 139.
(I describe below how this is done.) Suppose that the outcome of these executions
determine the value to be proposed in instances 135 and 140, but
leaves the proposed value unconstrained in all other instances. The leader
then executes phase 2 for instances 135 and 140, thereby choosing commands
135 and 140.
```

  新leader就是一个learner，需要最大可能知道已经被选择的命令。
  
假定他知道命令1-134,138，和139，意味着此一致性算法的实例1-134,138,139中的value被选择。(随后，我们将会发现命令序列中的空隙是如何产生的)。

紧接着，会执行135-137和所有大于139的实例的phase 1。(下面，我会描述这是如何做的)。

假如这些执行结果决定了在实例135和140中提议的value，但是保留其他实例中被提议的value不受限制。

leader接着会执行实例135和140的phase 2，因此选择命令135和140。

```c
  The leader, as well as any other server that learns all the commands
the leader knows, can now execute commands 1–135. However, it can’t
execute commands 138–140, which it also knows, because commands 136
and 137 have yet to be chosen. The leader could take the next two commands
requested by clients to be commands 136 and 137. Instead, we let it fill the
gap immediately by proposing, as commands 136 and 137, a special “noop”
command that leaves the state unchanged. (It does this by executing
phase 2 of instances 136 and 137 of the consensus algorithm.) Once these
no-op commands have been chosen, commands 138–140 can be executed.
```
  
  leader，以及学习了leader所知道的所有命令的其他服务，可以执行命令1-135。

然而，尽管他是知道命令138-140，但不能执行，因为命令136和137已经被选择。

leader执行的下两条客户端发送的命令将是命令136和137.紧接着，我们通过马上提议命令136和137，来填充间隙。

一个特殊的"noop"命令保证状态没有改变。(完成这个，通过执行实例136和137的phase 2)。

一旦这些no-op命令被选择，命令138-140可以执行了。

```c
  Commands 1–140 have now been chosen. The leader has also completed
phase 1 for all instances greater than 140 of the consensus algorithm, and it
is free to propose any value in phase 2 of those instances. It assigns command
number 141 to the next command requested by a client, proposing it as the
value in phase 2 of instance 141 of the consensus algorithm. It proposes the
next client command it receives as command 142, and so on.
```

  命令1-140已经被选择。leader完成了所有大于140实例的一致性算法的phase 1，在这些实例的phase 2，可以随意提议任意value。
  
它会把命令号141赋值给客户端请求的下一个命令，提议此命令作为实例141在一致性算法的Phase 2的value。

紧接着，把下一条客户端命令作为命令142，不断重复这个过程。

```c
  The leader can propose command 142 before it learns that its proposed
command 141 has been chosen. It’s possible for all the messages it sent
in proposing command 141 to be lost, and for command 142 to be chosen
before any other server has learned what the leader proposed as command
141. When the leader fails to receive the expected response to its phase 2
messages in instance 141, it will retransmit those messages. If all goes well,
its proposed command will be chosen. However, it could fail first, leaving a
gap in the sequence of chosen commands. In general, suppose a leader can
get α commands ahead—that is, it can propose commands i + 1 through
i +α after commands 1 through i are chosen. A gap of up to α−1 commands
could then arise.
```

  leader可以在学习被选择的命令141之前，提议命令142。leader很有可能已经提议了命令141，
  
  而其他服务还没来得及学习，此时很有可能提议的命令141中发送的消息被丢失，而命令142却被选择。
  
  当leader没有收到实例141在phase 2的预期的响应，他会重传这些消息。
  
  如果一切很顺利，他提议的命令会被选择。然而，很有可能首先失败了，在被选择的命令中留下一个间隙。
  
  一般情况下，假设领导者预先可以得到α命令，也就是说，可以在命令1到i被选择之后提议命令i + 1到i +α。 那么可能出现高达α-1个命令间隙。

```c  
  A newly chosen leader executes phase 1 for infinitely many instances
of the consensus algorithm—in the scenario above, for instances 135–137
and all instances greater than 139. Using the same proposal number for
all instances, it can do this by sending a single reasonably short message
to the other servers. In phase 1, an acceptor responds with more than a
simple OK only if it has already received a phase 2 message from some
proposer. (In the scenario, this was the case only for instances 135 and
140.) Thus, a server (acting as acceptor) can respond for all instances with
a single reasonably short message. Executing these infinitely many instances
of phase 1 therefore poses no problem.
```
  
  在上面的场景中，新选择的领导者为一致性算法的无限多个实例执行phase 1，有实例135-137和所有大于139的实例。
  
  对所有实例使用相同的提案号码，可以通过向其他服务器发送一条合理简短的消息来实现。
  
  一个acceptor只有在接收了来自一些proposer的phase 2消息，他才不会对phase 1的消息响应一个简单的OK。
  
  (这种场景下，这种情形对应的是实例135和140）。
  
  因此，一个服务(扮演了acceptor) 可以给所有的实例回应一个合理的短消息。
  
  执行无限多个实例的phase 1也不会有什么问题。

```c  
  Since failure of the leader and election of a new one should be rare
events, the effective cost of executing a state machine command—that is, of
achieving consensus on the command/value—is the cost of executing only
phase 2 of the consensus algorithm. It can be shown that phase 2 of the
Paxos consensus algorithm has the minimum possible cost of any algorithm
for reaching agreement in the presence of faults [2]. Hence, the Paxos algorithm
is essentially optimal.
```

  因为leader发生故障，而重新选择新leader是小概率事件，执行状态机命令的有效成本，也就是获取command/value的一致性，
  
  是仅仅执行一致性算法phase 2的成本。可以看出，当故障发生时，任何想要达到一致性的算法中，Paxos的代价是最小的。
  
  因此，Paxos算法基本上是最优的。

```c
  This discussion of the normal operation of the system assumes that there
is always a single leader, except for a brief period between the failure of the
current leader and the election of a new one. In abnormal circumstances,
the leader election might fail. If no server is acting as leader, then no new
commands will be proposed. If multiple servers think they are leaders, then
they can all propose values in the same instance of the consensus algorithm,
which could prevent any value from being chosen. However, safety is
preserved—two different servers will never disagree on the value chosen as
the ith state machine command. Election of a single leader is needed only
to ensure progress.
```

  关于系统正常运行的讨论，假定了总是有一个leader存在，除非是在当前leader故障，
  
而新leader选举期间，会出现无leader的情况。在异常情况下，leader选举会失败。

如果没有服务扮演leader，那么没有命令会被提议。如果有多个服务都是leader，那么他们都会在一致性算法的同一个实例中提议value，

这会导致任何value都不会被选择。然而，安全是能保证的，即，两个不同的服务器永远不会对第i个状态机命令选择的值产生不同意见。

选举一个单独的leader仅仅是为了确保进度。

```c
  If the set of servers can change, then there must be some way of determining
what servers implement what instances of the consensus algorithm.
The easiest way to do this is through the state machine itself. The current
set of servers can be made part of the state and can be changed with ordinary
state-machine commands. We can allow a leader to get α commands
ahead by letting the set of servers that execute instance i + α of the consensus
algorithm be specified by the state after execution of the ith state machine command. 
This permits a simple implementation of an arbitrarily sophisticated reconfiguration algorithm.
```

  如果服务器集合可以改变，那么肯定有办法决定哪些服务实现一致性算法的哪些实例。
  
实现这个目的最简单的方法就是通过状态机本身。当前的一组服务器可以作为状态的一部分，并可以使用普通的状态机命令进行更改。

### References

[1] Michael J. Fischer, Nancy Lynch, and Michael S. Paterson. Impossibility
of distributed consensus with one faulty process. Journal of the ACM,32(2):374–382, April 1985.

[2] Idit Keidar and Sergio Rajsbaum. On the cost of fault-tolerant consensus
when there are no faults—a tutorial. TechnicalReport MIT-LCS-TR-821,
Laboratory for Computer Science, Massachusetts Institute Technology,
Cambridge, MA, 02139, May 2001. also published in SIGACT News
32(2) (June 2001).

[3] Leslie Lamport. The implementation of reliable distributed multiprocess
systems. Computer Networks, 2:95–114, 1978.

[4] Leslie Lamport. Time, clocks, and the ordering of events in a distributed
system. Communications of the ACM, 21(7):558–565, July 1978.

[5] Leslie Lamport. The part-time parliament. ACM Transactions on Computer
Systems, 16(2):133–169, May 1998.
