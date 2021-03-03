# Ring of Fire


## Design

A ring shows the following characteristics:

- all nodes together create a closed circle of channels
- each node opens one channel to the next in a clockwise manner
- all channels within the ring are of the same channel size
- ideally nodes of higher capacity connect to nodes with lower capacity to distribute capacity along the ring
- channels are public
- all nodes are reachable through tor
- for easier rebalancing a service such as lightningto.me can be in the center of the ring
- optional: participants within the ring can create shortcuts between each other to make the ring more resilient

## Multiple rings
Rings can form connections. Preferably, the touchpoints of two rings should be the role of a reliable node runner with high capacity.

### Questions to answer

- Who is participating?
- What are their respective nodes' public key?
- How big are channel sizes within the ring?
- What is the order of each participant within the ring?

## How to open channels

Once the design is drafted out and each participant is aware of their position within the ring and knows who their partner is, we can start opening channels. Ideally, in coordination and at the same time.

Using LND, through the terminal we can use the command line interface to open a channel and push at the same time half of the amount to the next node.

```lncli openchannel --node_key [NODE_PUBLIC_KEY] --local_amt [AMOUNT_IN_SATS] --push_amt [AMOUNT_TO_PUSH] --sat_per_byte [FEE_SATS_PER_VBYTE] --min_confs 0```

- `NODE_PUBLIC_KEY`: the public key of the node of the partner you open the channel to
- `AMOUNT_IN_SATS`: for example 4000000
- `AMOUNT_TO_PUSH`: for example 2000000
- `FEE_SATS_PER_VBYTE`: for example 15
