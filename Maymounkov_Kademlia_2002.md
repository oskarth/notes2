Link: https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf

Raw notes.

What do I know beforehand?
It’s a distributed hash table with some nice properties. Efficient lookup?

What special properties does it have?

How does lookup work?
Each node has a 160 bit id (say, SHA1 of something). Lookup of a node is done in logarithmic number of steps and a single algorithm. It works like a binary tree: if we have a node 0010 then it must have some contact in each of those sub trees, e.g in 1xxx, 01xx, 001x. If we then look for 1101 it might know 101 in that sub tree. Then that node provides information (RPC-style) about nodes closer to the target node.

That’s for lookup. For storing a k-v pair is located at a node with ID that’s close to k.

XOR metric: to assign nodes k-v we need a specific node to place it at. This relies on a distance metric. By a XOR b we have a metric that fulfills basic metric criteria, like d(x,x)=0, else >0, x y ~ y x. Also triangle property, though no 100% sure why. Point is we can always find distance between nodes.

Example: 0010 xor 1110 = 1100, so quite far apart.

It’s also unidirectional, which means lookup path always convergences and we can do aggressive caching. Interesting property, what does this mean? There’s only one pair of nodes which have a set distance between them. I think that’s what it means. Trivial: dist xor a gives b. This is apparently not the case for some other DHTs. But not 100% clear on this, would need some illustration on this convergence.

Reason we couldn’t use this for enodes as well? Are we? Depends on what is stored I guess.

Each node also sends through its own id as identifier. 

Each node stored list of triples of IP/port (UDP)/node. For i between 0..160 based on distance in “k buckets”. Where k is a form of replication factor (?)

i=3, 000..111:
1..2, 2..4, 4..8

k bucket
000
001

k bucket
010
011

k bucket
100
101
110
111

As new state comes in (req or reply) it updates that k bucket with up to date contact info. It puts at end of list, but if bucket is full it checks first node in list to see if this live, and if live it rejects new info. This is a form of least-recently seen cache eviction. Proven to be very useful empirically in Gnutella study. Based on
- Lindy effect, old likely to stay around, means better uptime
- Can’t easily spam network with new bs nodes that are bad, since they would just get rejected

This is a great policy and seems super useful. Isn’t that a form of local reputation?

RPC calls: ping, store, find node, find value. Lookup k closest neighbors. Can do this concurrently with concurrency parameter, and then with new information keeping looking. Similar with store: to k, say 20, closest neighbors. If close neighbor doesn’t have value it can be instructed to store it as it should have it. Also require republishing etc to ensure live nodes have and data is useful. 

When finding a node reply should have k replies, nodes closest to target.

Republish kv pair every hour since nodes leave. (Why not use similar tactic for mail server and replicate?) Some optimizations to make sure tree is balanced.

For p2p it should be enough to know _any_ node, eg such as a friend, to discover rest of network. Good litmus test.

What about attacks? Mention of exponential back off for bad nodes.

And incentives for running this?

How is Ethereum different or same? How does our discovery protocol fit into this?

How can we use this distance metric and properties for replicating mail server?

And why not just random list of peers? Why this maxpeers etc nonsense, don’t get. Boot nodes etc different, but _why_?

Implications for mobile?

~90m