# Ring of Fire


## Design

A ring shows the following characteristics:

- all nodes together create a closed circle of channels
- each node opens one channel to the next in a clockwise manner
- all channels within the ring are of the same channel size
- ideally nodes of higher capacity connect to nodes with lower capacity to distribute capacity along the ring
- channels within the ring are either ALL private, or ALL public
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
- Should the channels be all public or all private?

## How to open channels

Once the design is drafted out and each participant is aware of their position within the ring and knows who their partner is, we can start opening channels. Ideally, in coordination and at the same time.

In order to ensure all channels that form the ring are balanced on inception, each node opens only one channel (minimising fees), and pushes 50% of the opened capacity to their connected node partner.

When all participants of the ring have opened channels, each node retains control of the total value of their opened channel.

### Using LND

#### lncli
We can use the command line interface to open a channel and push at the same time half of the amount to the next node.

```lncli openchannel --node_key [NODE_PUBLIC_KEY] --local_amt [AMOUNT_IN_SATS] --push_amt [AMOUNT_TO_PUSH] --sat_per_byte [FEE_SATS_PER_VBYTE] --min_confs 0```

- `NODE_PUBLIC_KEY`: the public key of the node of the partner you open the channel to
- `AMOUNT_IN_SATS`: for example 4000000
- `AMOUNT_TO_PUSH`: for example 2000000
- `FEE_SATS_PER_VBYTE`: for example 15

#### Thunderhub
Thunderhub which is available on MyNode and Umbrel allows for pushing half of the channel value onto the other side of the channel as part of the open. Using Thunderhub to open a channel, the "Push Tokens to Partner" option should be selected with the value "Half". This has the exact same effect as the CLI method for LND above.



## Migrating a node

If a participant finds the need to migrate the node to another instance. There are 2 ways to do so:

- copy the data folder and keep all channels
- starting from scratch

For LND, you can find the documentation here: https://github.com/lightningnetwork/lnd/blob/master/docs/safety.md#migrating-a-node-to-a-new-device

## Switching from clearnet to Tor

If the members of the ring opt for Tor only but a participants finds themself still on clearnet, the following guide can be followed for LND:
https://github.com/lightningnetwork/lnd/blob/master/docs/safety.md#migrating-a-node-from-clearnet-to-tor

## LN Maintenance

### Keep backups

// TODO

### Preventing data corruption
Sudden blackouts can be devastating to your data. LN nodes are constantly chatting with other nodes, this means they often update their database. If a node is interrupted in the middle of writing data, it might become corrupted. Should this happen, the safest way to recover your funds might only be to close all channels from your channel backup.
In order to avoid this situation, one should consider plugging a UPS (Uninterruptible Power Supply) inbetween.

Here's a good discussion about it: https://github.com/rootzoll/raspiblitz/issues/263#issue-397855893

If you already have or you are going to purchase a UPS without shutdown signal, you can use the following script as a workaround: https://github.com/Czino/graceful-shutdown
