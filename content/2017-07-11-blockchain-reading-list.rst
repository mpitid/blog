
:title: A blockchain reading list
:date: 2017-07-07 16:00
:tags: blockchain, distributed systems
:status: draft

I've often struggled to give good reference material when talking to
people about the blockchain. The landscape is filled with obscure
buzzwords and overzealous hype and making one's way through it is a
challenge. This post is my attempt to provide a map, or rather a map of
maps [1]_. I will start with some academic papers on distributed
consensus before moving into more dangerous waters towards the end.

.. figure:: {filename}/images/excuse-me-blockchain.jpg
   :width: 300
   :scale: 50%
   :figwidth: 200
   :align: center
   :alt: talking about the blockchain

For anyone interested in distributed systems, I recommend starting with
`The quest for a scalable blockchain fabric: Proof-of-work vs. BFT
replication`_. It's probably the best overview paper I've come across so
far. It describes key concepts --blockchains/distributed ledgers,
bitcoin, ethereum, proof-of-work (PoW), smart contracts, state machine
replication (SMR) and crash fault-tolerant (CFT) vs Byzantine
fault-tolerant (BFT) consensus protocols. It then weaves them into a
wider narrative where PoW based consensus --as implemented in public
blockchains-- is compared with traditional CFT/BFT solutions to
consensus --which is where so-called permissioned blockchains are
turning their attention.

Moreover, an analysis of blockchains is provided in terms
of their consensus algorithm and the implications this choice has in
aspects ranging from identity management to consensus finality, scalability,
performance (latency, throughput, power consumption), tolerated power of
adversary, network synchrony assumptions, and existence of correctness
proofs of the underlying protocols.

PoW-based blockchains and BFT consensus algorithms are placed at
opposite ends of the scalability-vs-performance spectrum: PoW offers
high scalability but poor performance, whereas current BFT (and CFT)
consensus algorithms offer good performance but their scalability is
poorly understood and they are seldom deployed with more than a handful
of replicas [2]_. The paper then proceeds to discuss current efforts and
suggestions for improving blockchain scalability, before concluding with
a short list of open problems.

The major contribution in my opinion is its function as a curated and
comprehensive list of references which cover all of the above aspects in
detail for anyone looking to dive deeper [3]_. It also provides an
informed but sober --though not completely unbiased-- overview of the
current state of affairs, from an industrial research perspective [4]_.
A good follow-up paper is `Rethinking Permissioned Blockchains`_ which
explores the author's proposal for the redesign of the architecture of
`HyperLedger Fabric`_.

All references in that paper are worth looking into, but I would
strongly recommend `State Machine Replication for the Masses with
BFT-SMaRt`_ for anyone interested in the *practicalities* of
*implementing* a BFT consensus algorithm. The full technical report
describes the design and implementation of a production BFT consensus
algorithm in Java, available under an open source license. The
architecture is modular, the types of failure tolerated are configurable
--from crashes to malicious Byzantine nodes-- and the set of replicas
can be adjusted dynamically while the system is running --something
often missing from available BFT implementations.

The implementation's API and programming model are discussed, and the
system is evaluated against alternative BFT implementations (PBFT and
UpRight) as well as CFT ones (JPaxos). Finally, key take-aways from
building and maintaining the system are provided.

This is an excellent starting point for anyone interested in
implementing a *practical* distributed consensus system, be it a CFT or
BFT one. The main contribution in my opinion is the lessons and
take-aways from developing and maintaining the system over the course of
many years and across changing teams. The description of how the
low-level design was tackled and how the system was tested and evaluated
is also very useful.

For anyone with a further interest in distributed systems and
particularly consensus protocols, `Can't we all just agree`_ is a series
of posts from Adrian Colyer's `morning paper`_ blog and a good starting
point. It covers some classical CFT consensus protocols such as
Viewstamped Replication, Paxos, Zab and Raft in the form of paper
summaries. Studying at least one of those designs in more detail is
highly recommended; Raft is probably the easiest one and also `widely
implemented`_. A useful --and often entertaining-- companion resource in
this area is `Kyle Kingsbury's blog`_ on distributed systems, often
elaborating on how many of them fail to provide their touted guarantees
in practice.


So far we have focused mostly on academic issues in distributed systems.
For those preferring less formalised descriptions, the `Ethereum
whitepaper`_ works as a good introduction to both Bitcoin and Ethereum,
the motivation and technical decisions behind it and some of the
potential applications.

But what about all the business hype behind the blockchain? This is
where things get tricky. If you have any exposure to the blockchain
space and the companies in it, you've probably come across one or two
*technology whitepapers* [5]_. Their typical promise is infinite
scalability, seamless interoperability and an end to world hunger --but
you can only choose two at a time, we are realists after all. The
problem with many whitepapers is that they are mostly marketing material
dressed in technical jargon. They use impressive-sounding language and
give out just enough to hint at a competitive advantage, but not enough
to reveal its true extent --or lack thereof. This makes it challenging
to distinguish between serious efforts and good ol' snake oil [6]_.

If you want to explore this area further --despite these warnings-- I
recommend taking a look at the `blog of George Samman`_ which has
summarised a lot of these efforts, mostly in the financial sector. I'll
admit I am intrigued by Kadena (based on Juno_ which was itself based on
a BFT extension of Raft) and HashGraph (a non-deterministic BFT
consensus algorithm), although the veracity of some of the claims made
remains to be verified.

Finally, if your buzzword tolerance is high enough [7]_, `Blockchains
and Smart Contracts for the Internet of Things`_ may serve as an
introduction into other popular areas of potential application. The
report provides a blockchain taxonomy, examines the potential benefits
and drawbacks for IoT and hints at one of the other major open problems
in the area besides distributed consensus: privacy and confidentiality
of transactions [8]_. The reference section is extensive, linking to a
plethora of websites and blogs in the wider blockchain space.

.. Footnotes

.. [1] Alas, the map is never the territory.

.. [2] Note however that scalability should not always be a `goal in
   itself`_ and permissioned chains don't necessarily need to scale to
   hundreds of thousands of nodes.

.. [3] Also major  points for citing some vintage `James Mickens`_! :D

.. [4] Marko Vukolić works for IBM Research, is actively involved with
   the HyperLedger_ project and has done extensive research around
   Byzantine consensus.

.. [5] One would think that whitepapers have become a prerequisite to
   filing papers of incorporation!

.. [6] But see `Attack of the 50-foot blockchain`_ for a colourfully
   illustrated guide.

.. [7] Remember that self-driving deep-neural 3D-printed drone-cars will
   one day run on the post-quantum semantic blockchain, they will be
   beautiful and all-encompassing, guiding us serenely into the
   singularity we've all been waiting for so long!

.. [8] A third one is interoperability, both with legacy systems and
   other blockchains.

.. Links

.. _The quest for a scalable blockchain fabric\: Proof-of-work vs. BFT replication: https://vukolic.github.io/iNetSec_2015.pdf
.. _Rethinking permissioned blockchains: https://vukolic.github.io/rethinking-permissioned-blockchains-BCC2017.pdf
.. _State Machine Replication for the Masses with BFT-SMaRt: http://repositorio.ul.pt/bitstream/10455/6897/1/TR-2013-07.pdf
.. _Blockchains and Smart Contracts for the Internet of Things: http://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=7467408

.. _Can't we all just agree: https://blog.acolyer.org/2015/03/01/cant-we-all-just-agree/
.. _morning paper: https://blog.acolyer.org/
.. _Ethereum whitepaper: https://github.com/ethereum/wiki/wiki/White-Paper
.. _goal in itself: http://www.frankmcsherry.org/graph/scalability/cost/2015/01/15/COST.html
.. _James Mickens: http://scholar.harvard.edu/files/mickens/files/thesaddestmoment.pdf
.. _HyperLedger: https://www.hyperledger.org
.. _HyperLedger Fabric: https://github.com/hyperledger/fabric
.. _Attack of the 50-foot blockchain: https://davidgerard.co.uk/blockchain/
.. _widely implemented: https://raft.github.io/
.. _blog of George Samman: http://sammantics.com
.. _Juno: https://github.com/kadena-io/juno
.. _Kyle Kingsbury's blog: https://aphyr.com/tags/Distributed-Systems

.. vim: set tw=72:
