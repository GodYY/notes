[TOC]

# 1. 起源		

​		CAP原本是一个猜想，2000年PODC大会的时候大牛Brewer提出的，他认为在设计一个大规模可扩放的网络服务时候会遇到三个特性：一致性（consistency）、可用性（Availability）、分区容错（partition-tolerance）都需要的情景，然而这是不可能都实现的。之后在2003年的时候，Mit的Gilbert和Lynch就正式的证明了这三个特征确实是不可以兼得的。该理论是NoSQL数据库管理系统构建的基础。。

　　Consistency、Availability、Partition-tolerance的提法是由Brewer提出的，而Gilbert和Lynch在证明的过程中改变了Consistency的概念，将其转化为Atomic。Gilbert认为这里所说的Consistency其实就是数据库系统中提到的ACID的另一种表述：

　　一个用户请求要么成功、要么失败，不能处于中间状态（Atomic）；

　　一旦一个事务完成，将来的所有事务都必须基于这个完成后的状态（Consistent）；

　　未完成的事务不会互相影响（Isolated）；

　　一旦一个事务完成，就是持久的（Durable）。

　　对于Availability，其概念没有变化，指的是对于一个系统而言，所有的请求都应该‘成功’并且收到‘返回’。

　　对于Partition-tolerance，所指就是分布式系统的容错性。节点crash或者网络分片都不应该导致一个分布式系统停止服务。

# 2. CAP 简介

　　CAP定律说的是在一个分布式计算机系统中，一致性，可用性和分区容错性这三种保证无法同时得到满足，最多满足两个。

## 2.1 强一致性

　　强一致性：系统在执行过某项操作后仍然处于一致的状态。在分布式系统中，更新操作执行成功后所有的用户都应该读到最新的值，这样的系统被认为是具有强一致性的。 等同于所有节点访问同一份最新的数据副本；

　　All clients always have the same view of the data。

## 2.2 可用性

​	可用性：每一个操作总是能够在一定的时间内返回结果，这里需要注意的是"一定时间内"和"返回结果"。一定时间指的是，在可以容忍的范围内返回结果，结果可以是成功或者失败。 对数据更新具备高可用性（A）；

　　Each client can always read and write。

## 2.3 分区容错性

　　分区容错性：理解为在存在网络分区的情况下，仍然可以接受请求（满足一致性或可用性)。这里的网络分区是指由于某种原因，网络被分成若干个孤立的区域，而区域之间互不相通。还有一些人将分区容错性理解为系统对节点动态加入和离开的能力，因为节点的加入和离开可以认为是集群内部的网络分区。

​		Partition Tolerance的意思是，在网络中断，消息丢失的情况下，系统照样能够工作。 以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

## 2.4 放弃 C/A/P

　　放弃P：如果想避免分区容错性问题的发生，一种做法是将所有的数据（与事务相关的）都放在一台机器上。虽然无法100%保证系统不会出错，但不会碰到由分区带来的负面效果。当然这个选择会严重的影响系统的扩展性。

　　放弃A:相对于放弃“分区容错性“来说，其反面就是放弃可用性。一旦遇到分区容错故障，那么受到影响的服务需要等待一定的时间，因此在等待期间系统无法对外提供服务。

　　放弃C：这里所说的放弃一致性，并不是完全放弃数据一致性，而是放弃数据的强一致性，而保留数据的最终一致性。以网络购物为例，对只剩下一件库存的商品，如果同时接受到了两份订单，那么较晚的订单将被告知商品告罄。

​		一致性与可用性的决择： 而CAP理论就是说在分布式存储系统中，最多只能实现上面的两点。而由于当前的网络硬件肯定 会出现延迟丢包等问题，所以分区容忍性是我们必须需要实现的。所以我们只能在一致性和可用 性之间进行权衡。

# 3. 基本 CAP 的证明思路

　　CAP的证明基于异步网络，异步网络也是反映了真实网络中情况的模型。真实的网络系统中，节点之间不可能保持同步，即便是时钟也不可能保持同步，所有的节点依靠获得的消息来进行本地计算和通讯。这个概念其实是相当强的，意味着任何超时判断也是不可能的，因为没有共同的时间标准。之后我们会扩展CAP的证明到弱一点的异步网络中，这个网络中时钟不完全一致，但是时钟运行的步调是一致的，这种系统是允许节点做超时判断的。

　　CAP的证明很简单，假设两个节点集{G1, G2}，由于网络分片导致G1和G2之间所有的通讯都断开了，如果在G1中写，在G2中读刚写的数据， G2中返回的值不可能为G1中的写值。由于A的要求，G2一定要返回这次读请求，由于P的存在，导致C一定是不可满足的。

# 4. CAP的理解

## 4.2 形式化描述

　　要真正理解 CAP 理论必须要读懂它的形式化描述。 形式化描述中最重要的莫过于对 Consistency, Availability, Partition-tolerance 的准确定义。

　　Consistency (一致性) 实际上等同于系统领域的 before-or-after atomicity 这个术语，或者等同于 linearizable (可串行化) 这个术语。具体来说，系统中对一个数据的读和写虽然包含多个子步骤并且会持续一段时间才能执行完，但是在调用者看来，读操作和写操作都必须是单个的即时完成的操作，不存在重叠。对一个写操作，如果系统返回了成功，那么之后到达的读请求都必须读到这个新的数据；如果系统返回失败，那么所有的读，无论是之后发起的，还是和写同时发起的，都不能读到这个数据。

　　要说清楚 Availability 和 Partition-tolerance 必须要定义好系统的故障模型。在形式化证明中，系统包含多个节点，每个节点可以接收读和写的请求，返回成功或失败，对读还要返回一个数据。和调用者之间的连接是不会中断的，系统的节点也不会失效，唯一的故障就是报文的丢失。 Partition-tolerance 指系统中会任意的丢失报文(这和“最终会有一个报文会到达”是相对的)。 Availability 是指所有的读和写都必须要能终止。

　　注： “Availability 是指所有的读和写都必须要能终止” 这句话听上去很奇怪，为什么不是“Availability 是指所有的写和读都必须成功”？ 要回答这个问题，我们可以仔细思考下“什么是成功”。“成功”必须要相对于某个参照而言，这里的参照就是 Consistency。

## 4.3 两种重要的分布式场景

　　关于对CAP理论中一致性C的理解，除了上述数据副本之间的读写一致性以外，分布式环境中还有两种非常重要的场景，如果不对它们进行认识与讨论，就永远无法全面地理解CAP，当然也就无法根据CAP做出正确的解释。

### 4.3.1 分布式环境中的事务场景    

　　我们知道，在关系型数据库的事务操作遵循ACID原则，其中的一致性C，主要是指一个事务中相关联的数据在事务操作结束后是一致的。所谓ACID原则，是指在写入/异动资料的过程中，为保证交易正确可靠所必须具备的四个特性：即原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）和持久性（Durability）。

　　例如银行的一个存款交易事务，将导致交易流水表增加一条记录。同时，必须导致账户表余额发生变化，这两个操作必须是一个事务中全部完成，保证相关数据的一致性。而前文解释的CAP理论中的C是指对一个数据多个备份的读写一致性。表面上看，这两者不是一回事，但实际上，却是本质基本相同的事物：数据请求会等待多个相关数据操作全部完成才返回。对分布式系统来讲，这就是我们通常所说的分布式事务问题。

　　众所周知，分布式事务一般采用两阶段提交策略来实现，这是一个非常耗时的复杂过程，会严重影响系统效率，在实践中我们尽量避免使用它。在实践过程中，如果我们为了扩展数据容量将数据分布式存储，而事务的要求又完全不能降低。那么，系统的可用性一定会大大降低，在现实中我们一般都采用对这些数据不分散存储的策略。

　　当然，我们也可以说，最常使用的关系型数据库，因为这个原因，扩展性（分区可容忍性P）受到了限制，这是完全符合CAP理论的。但同时我们应该意识到，这对NoSQL数据库也是一样的。如果NoSQL数据库也要求严格的分布式事务功能，情况并不会比关系型数据库好多少。只是在NoSQL的设计中，我们往往会弱化甚至去除事务的功能，该问题才表现得不那么明显而已。

　　因此，在扩展性问题上，如果要说关系型数据库是为了保证C、A而牺牲P，在尽量避免分布式事务这一点上来看，应该是正确的。也就是说：关系型数据库应该具有强大的事务功能，如果分区扩展，可用性就会降低；而NoSQL数据库干脆弱化甚至去除了事务功能，因此，分区的可扩展性就大大增加了。

### 4.3.2 分布式环境中的关联场景    

　　初看起来，关系型数据库中常用的多表关联操作与CAP理论就更加不沾边了。但仔细考虑，也可以用它来解释数据库分区扩展对关联所带来的影响。对一个数据库来讲，采用了分区扩展策略来扩充容量，数据分散存储了，很显然多表关联的性能就会下降，因为我们必须在网络上进行大量的数据迁移操作，这与CAP理论中数据副本之间的同步操作本质上也是相同的。

　　因此，如果要保证系统的高可用性，需要同时实现强大的多表关系操作的关系型数据库在分区可扩展性上就遇到了极大的限制（即使是那些采用了各种优秀解决方案的MPP架构的关系型数据库，如TeraData，Netezza等，其水平可扩展性也是远远不如NoSQL数据库的），而NoSQL数据库则干脆在设计上弱化甚至去除了多表关联操作。那么，从这一点上来理解"NoSQL数据库是为了保证A与P，而牺牲C"的说法，也是可以讲得通的。当然，我们应该理解，关联问题在很多情况下不是并行处理的优点所在，这在很大程度上与Amdahl定律相符合。

　　所以，从事务与关联的角度来看关系型数据库的分区可扩展性为什么受限的原因是最为清楚的。而NoSQL数据库也正是因为弱化，甚至去除了像事务与关联（全面地讲，其实还有索引等特性）等在分布式环境中会严重影响系统可用性的功能，才获得了更好的水平可扩展性。

　　那么，如果将事务与关联也纳入CAP理论中一致性C的范畴的话，问题就很清楚了：关于“关系型数据库为了保证一致性C与可用性A，而不得不牺牲分区可容忍性P”的说法便是正确的了。但关于"NoSQL选择了C与P，或者A与P"的说法则是错误的，所有的NoSQL数据库在设计策略的大方向上都是选择了A与P（虽然对同一数据多个副本的读写一致性问题的设计各有不同），从来没有完全选择C与P的情况存在。

　　现在看来，如果理解CAP理论只是指多个数据副本之间读写一致性的问题，那么它对关系型数据库与NoSQL数据库来讲是完全一样的，它只是运行在分布式环境中的数据管理设施在设计读写一致性问题时需要遵循的一个原则而已，却并不是NoSQL数据库具有优秀的水平可扩展性的真正原因。而如果将CAP理论中的一致性C理解为读写一致性、事务与关联操作的综合，则可以认为关系型数据库选择了C与A，而NoSQL数据库则全都是选择了A与P，但并没有选择C与P的情况存在。

# 5. 一致性分类

　　对于分布式数据系统，分区容忍性是基本要求，否则就失去了价值。因此设计分布式数据系统，就是在一致性和可用性之间取一个平衡。对于大多数WEB应用，其实并不需要强一致性，因此牺牲一致性而换取高可用性，是多数分布式数据库产品的方向。

　　当然，牺牲一致性，并不是完全不管数据的一致性，否则数据是混乱的，那么系统可用性再高分布式再好也没有了价值。牺牲一致性，只是不再要求关系型数据库中的强一致性，而是只要系统能达到最终一致性即可，考虑到客户体验，这个最终一致的时间窗口，要尽可能的对用户透明，也就是需要保障“用户感知到的一致性”。通常是通过数据的多份异步复制来实现系统的高可用和数据的最终一致性的，“用户感知到的一致性”的时间窗口则取决于数据复制到一致状态的时间。         

　　对于一致性，可以分为从客户端和服务端两个不同的视角。从客户端来看，一致性主要指的是多并发访问时更新过的数据如何获取的问题。从服务端来看，则是更新如何复制分布到整个系统，以保证数据最终一致。一致性是因为有并发读写才有的问题，因此在理解一致性的问题时，一定要注意结合考虑并发读写的场景。

## 5.1 客户端角度

　　从客户端角度，多进程并发访问时，更新过的数据在不同进程如何获取的不同策略，决定了不同的一致性。对于关系型数据库， 要求更新过的数据能被后续的访问都能看到，这是**强一致性**。如果能容忍后续的部分或者全部访问不到，则是**弱一致性**。如果经过一段时间后要求能访问到更新后的数据，则是**最终一致性**。

　　最终一致性根据更新数据后各进程访问到数据的时间和方式的不同，又可以区分为：

- 因果一致性(CAUSAL CONSISTENCY)：如果进程A通知进程B它已更新了一个数据项，那么进程B的后续访问将返回更新后的值，且一次写入将保证取代前一次写入。与进程A无因果关系的进程C的访问遵守一般的最终一致性规则。
- 读己之所写（READ-YOUR-WRITES）一致性：当进程A自己更新一个数据项之后，它总是访问到更新过的值，绝不会看到旧值。这是因果一致性模型的一个特例。
- 会话（SESSION）一致性：这是上一个模型的实用版本，它把访问存储系统的进程放到会话的上下文中。只要会话还存在，系统就保证“读己之所写”一致性。如果由于某些失败情形令会话终止，就要建立新的会话，而且系统的保证不会延续到新的会话。
- 单调（MONOTONIC）读一致性：如果进程已经看到过数据对象的某个值，那么任何后续访问都不会返回在那个值之前的值。
- 单调写一致性：系统保证来自同一个进程的写操作顺序执行。要是系统不能保证这种程度的一致性，就非常难以编程了。



​		上述最终一致性的不同方式可以进行组合，例如单调读一致性和读己之所写一致性就可以组合实现。并且从实践的角度来看，这两者的组合，读取自己更新的数据，和一旦读取到最新的版本不会再读取旧版本，对于此架构上的程序开发来说，会少很多额外的烦恼（读自己：保证强一致性；读别人：最终一致性）。

## 5.2 服务端角度

　　从服务端角度，如何尽快将更新后的数据分布到整个系统，降低达到最终一致性的时间窗口，是提高系统的可用度和用户体验非常重要的方面。

​		对于分布式数据系统：N — 数据复制的份数，W — 更新数据时需要保证写完成的节点数，R — 读取数据的时候需要读取的节点数。如果W+R>N，写的节点和读的节点重叠，则是强一致性。例如对于典型的一主一备同步复制的关系型数据库，N=2,W=2,R=1，则不管读的是主库还是备库的数据，都是一致的。如果W+R<=N，则是弱一致性。例如对于一主一备异步复制的关系型数据库，N=2,W=1,R=1，则如果读的是备库，就可能无法读取主库已经更新过的数据，所以是弱一致性。

​		对于分布式系统，为了保证高可用性，一般设置N>=3。不同的N,W,R组合，是在可用性和一致性之间取一个平衡，以适应不同的应用场景。如果N=W,R=1，任何一个写节点失效，都会导致写失败，因此可用性会降低，但是由于数据分布的N个节点是同步写入的，因此可以保证强一致性。如果N=R,W=1，只需要一个节点写入成功即可，写性能和可用性都比较高。但是读取其他节点的进程可能不能获取更新后的数据，因此是弱一致性。这种情况下，如果W<(N+1)/2，并且写入的节点不重叠的话，则会存在写冲突（？）。

# 6. 传统数据库与NoSQL数据库

​		传统的关系型数据库在功能支持上通常很宽泛，从简单的键值查询，到复杂的多表联合查询再到事务机制的支持。而与之不同的是，NoSQL系统通常注重性能和扩展性，而非事务机制（事务就是强一致性的体现）。 

​		传统的SQL数据库的事务通常都是支持 ACID 的强事务机制。

​		NoSQL系统仅提供对行级别的原子性保证，也就是说同时对同一个Key下的数据进行的两个操作，在实际执行的时候是会串行的执行，保证了每一个Key-Value对不会被破坏。例如 MongoDB，它是不支持事务机制的，同时也不提倡多表关联的复杂模式设计，它只保证对单个文档(相当于关系数据库中的记录)读写的原子性。

​		补充:  MPP架构介绍 MPP (Massively Parallel Processing)，大规模并行处理系统，这样的系统是由许多松耦合的处理单元组成的，要注意的是这里指的是处理单元而不是处理器。每个单元内的CPU都有自己私有的资源，如总线，内存，硬盘等。在每个单元内都有操作系统和管理数据库的实例复本。这种结构最大的特点在于不共享资源。

# 7. 战胜 CAP

​		核心内容就是放松Gilbert和Lynch证明中的限制：“系统必须同时达到CAP三个属性”，放松到“系统可以不同时达到CAP，而是分时达到”。 

​		CAP理论被很多人拿来作为分布式系统设计的金律，然而感觉大家对CAP这三个属性的认识却存在不少误区。从CAP的证明中可以看出来，这个理论的成立是需要很明确的对C、A、P三个概念进行界定的前提下的。在本文中笔者希望可以对论文和一些参考资料进行总结并附带一些思考。

　　CAP理论的表述很好地服务了它的目的，即开阔设计师的思路，在多样化的取舍方案下设计出多样化的系统。在过去的十几年里确实涌现了不计其数的新系统，也随之在数据一致性和可用性的相对关系上产生了相当多的争论。“三选二”的公式一直存在着误导性，它会过分简单化各性质之间的相互关系。现在我们有必要辨析其中的细节。实际上只有“在分区存在的前提下呈现完美的数据一致性和可用性”这种很少见的情况是CAP理论不允许出现的。

​		虽然设计师仍然需要在分区的前提下对数据一致性和可用性做取舍，但具体如何处理分区和恢复一致性，这里面有不计其数的变通方案和灵活度。当代CAP实践应将目标定为针对具体的应用，在合理范围内最大化数据一致性和可用性的“合力”。这样的思路延伸为如何规划分区期间的操作和分区之后的恢复，从而启发设计师加深对CAP的认识，突破过去由于CAP理论的表述而产生的思维局限。

## 7.1 为什么“三选二”公式有误导性

​		理解CAP理论的最简单方式是想象两个节点分处分区两侧。允许至少一个节点更新状态会导致数据不一致，即丧失了C性质。如果为了保证数据一致性，将分区一侧的节点设置为不可用，那么又丧失了A性质。除非两个节点可以互相通信，才能既保证C又保证A，这又会导致丧失P性质。一般来说跨区域的系统，设计师无法舍弃P性质，那么就只能在数据一致性和可用性上做一个艰难选择。不确切地说，NoSQL运动的主题其实是创造各种可用性优先、数据一致性其次的方案；而传统数据库坚守ACID特性（原子性、一致性、隔离性、持久性），做的是相反的事情。

​		“三选二”的观点在几个方面起了误导作用。首先，由于分区很少发生，那么在系统不存在分区的情况下没什么理由牺牲C或A。其次，C与A之间的取舍可以在同一系统内以非常细小的粒度反复发生，而每一次的决策可能因为具体的操作，乃至因为牵涉到特定的数据或用户而有所不同。最后，这三种性质都可以在程度上衡量，并不是非黑即白的有或无。可用性显然是在0%到100%之间连续变化的，一致性分很多级别，连分区也可以细分为不同含义，如系统内的不同部分对于是否存在分区可以有不一样的认知。

　　要探索这些细微的差别，就要突破传统的分区处理方式，而这是一项根本性的挑战。因为分区很少出现，CAP在大多数时候允许完美的C和A。但当分区存在或可感知其影响的情况下，就要预备一种策略去探知分区并显式处理其影响。这样的策略应分为三个步骤：探知分区发生，进入显式的分区模式以限制某些操作，启动恢复过程以恢复数据一致性并补偿分区期间发生的错误。

## 7.2 解决 CAP

　　根据一些专家的分析，CAP并不是一个严谨的定律，并不是牺牲了Consistency，就一定能同时获得Availability和Partition Tolerance。还有一个很重要的因素是Latency，在CAP中并没有体现。在现在NoSQL以及其他一些大规模设计时，A和P并不是牺牲C或部分牺牲C的借口，因为即使牺牲了C，也不一定A和P，并且C不一定必须要牺牲。

　　淘宝一天就处理了1亿零580万，而12306一天处理的交易仅仅166万条 ，如果从并发性上来说，淘宝的并发量远比12306大，但天猫的商品信息，促销数据都可以做缓存，做[CDN](https://cloud.tencent.com/product/cdn?from=10680)，而12306的“商品”是一个个座位，这些座位必须通过后端数据库即时查询出来，状态的一致性要求很高。

　　从这点上看，12306的商品信息很难利用到缓存，因此12306查看“商品”的代价是比较大的，涉及到一系列的后端数据库操作，从这个角度讲，12306的复杂度是高于天猫的。 淘宝的商品相对独立，而12306商品之间的关联性很大，由于CAP定律限制，如果其商品的一致性要求过高，必然对可用性和分区容错性造成影响。

　　因此，业务设计上，如果找到一条降低一致性要求时，还能保证业务的正确性的业务分拆之路。举个例子，火车票查询时，不要显示多少张，而是显示“有”或“无”，或者显示>100张，50~100,小于50等，这样就可以减小状态的更新频率，充分使用缓存数据。

　　CAP 理论说在一个系统中对某个数据不存在一个算法同时满足 Consistency, Availability, Partition-tolerance。注意，这里边最重要和最容易被人忽视的是限定词“对某个数据不存在一个算法”。这就是说在一个系统中，可以对某些数据做到 CP, 对另一些数据做到 AP，就算是对同一个数据，调用者可以指定不同的算法，某些算法可以做到 CP，某些算法可以做到 AP。

## 7.3 做到两项

　　要做到 CP， 系统可以把这个数据只放在一个节点上，其他节点收到请求后向这个节点读或写数据，并返回结果。很显然，串行化是保证的。但是如果报文可以任意丢失的话，接受请求的节点就可能永远不返回结果。

　　要做到 CA， 一个现实的例子就是单点的数据库。你可能会疑惑“数据库也不是 100% 可用的呀？” 要回答这个疑惑，注意上面说的故障模型和 availability 的定义就可以了。

​		要做到 AP， 系统只要每次对写都返回成功，对读都返回固定的某个值就可以了。

​		如果我们到这里就觉得已近掌握好 CAP 理论了，那么就相当于刚把橘子剥开，就把它扔了。

​		CAP 理论更重要的一个结果是， 在 Partial Synchronous System (半同步系统) 中，一个弱化的 CAP 是能达到的:对所有的数据访问，总返回一个结果 * 如果期间没有报文丢失，那么返回一个满足 consistency 要求的结果。

　　这里的半同步系统指每个节点存在一个时钟，这些时钟不需要同步，但是按照相同的速率流逝。更通俗的来说，就是一个能够实现超时机制的系统。

　　举个例子，系统可以把这个数据只放在一个节点上，其他节点收到请求后向这个节点读或写数据，并设置一个定时器，如果超时前得到结果，那么返回这个结果，否则返回失败。更进一步的，也是最重要的，实现一个满足最终一致性 (Eventually Consistency) 和 AP 的系统是可行的。 现实中的一个例子是 Cassandra 系统。

　　而对于分布式数据系统，分区容忍性是基本要求，否则就失去了价值。因此设计分布式数据系统，就是在一致性和可用性之间取一个平衡。对于大多数WEB应用，其实并不需要强一致性，因此牺牲一致性而换取高可用性，是多数分布式数据库产品的方向。 当然，牺牲一致性，并不是完全不管数据的一致性，否则数据是混乱的，那么系统可用性再高分布式再好也没有了价值。牺牲一致性，只是不再要求关系型数据库中的强一致性，而是只要系统能达到最终一致性即可，考虑到客户体验，这个最终一致的时间窗口，要尽可能的对用户透明，也就是需要保障“用户感知到的一致性”。通常是通过数据的多份异步复制来实现系统的高可用和数据的最终一致性的，“用户感知到的一致性”的时间窗口则取决于数据复制到一致状态的时间。
