# Ring of Fire

Lightning is based on cooperation, but it is common that channel openings happen one-sided, (i. e. you are not in direct contact to your channel partner).

To strengthen cooperation and form strong partnerships, we have devised the concept of a ring of nodes, we dub *The Ring of Fire*. It serves as a template to bring node runners together and lift each other up.

## Benefits of participating in a Ring

Working on a common goal forms communities. If complications arise, participants can quickly reach out and receive help.

The connectivity of each node increases significicantly as every other participants adds their channels to the outside.

We also currently investitage how and which data we share within the ring. This insight could give ring operators an edge over the rest of the users within the lightning network.

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

### Multiple rings
Rings can form connections. Preferably, the touchpoints of two rings should be the role of a reliable node runner with high capacity.

### Questions to answer

- Who is participating?
- What are their respective nodes' public key?
- How big are channel sizes within the ring?
- What is the order of each participant within the ring?
- Should the channels be all public or all private?

### How to open channels

Once the design is drafted out and each participant is aware of their position within the ring and knows who their partner is, we can start opening channels. Ideally, in coordination and at the same time. To minimise fees, each node opens only one channel to the next.

When all participants of the ring have opened channels, each node retains control of the total value of their opened channel.

#### Using LND

##### lncli
We can use the command line interface to open a channel and push at the same time half of the amount to the next node.

```lncli openchannel --node_key [NODE_PUBLIC_KEY] --local_amt [AMOUNT_IN_SATS] --sat_per_byte [FEE_SATS_PER_VBYTE] --min_confs 0```

- `NODE_PUBLIC_KEY`: the public key of the node of the partner you open the channel to
- `AMOUNT_IN_SATS`: for example 4000000
- `FEE_SATS_PER_VBYTE`: for example 15

##### RTL
- Navigate to *Lightning > Peers/Channels > Peers*
- Click on *Add Peer*
- Paste in the Lightning Address of your peer
- Add Peer

- Type in desired amount in sats
- choose appropriate fee rate
- Click on *Open Channel*

### The Initial Balancing Act

Once the ring is closed (i.e. all channels are opened), one participant can create an invoice of half of the channel capacity. The participant will pay the invoice to themselves by building a route across the ring.

Routes can be manually built by using the following tool

#### lncli

##### Build the route
First test if the route can be built. If a node is offline of the parameters are not correct, you will get informed. If successfull, you will be presented with the information about the payment route, including the fees you will pay.

```lncli buildroute --amt [AMOUNT_IN_SATS] --hops [LIST_OF_PUBLIC_KEYS_OF_PEERS] --outgoing_chan_id [OUTGOING_CHAN_ID]```

- `AMOUNT_IN_SATS`: for example 2000000
- `LIST_OF_PUBLIC_KEYS_OF_PEERS`: comma separated list of public keys of peers in order of the position in the ring
- `OUTGOING_CHAN_ID`: the ID of the outgoing channel you start with

##### Pay yourself
Create an invoice of half of the channel capacity

Find the payment hash (not to be confused with the invoice)

*Find out the payment hash in lncli*

Use the command `lncli listinvoices` and read `r_hash` value of the corresponding invoice.

You can then use the payment hash to plug into the buildroute command

```lncli buildroute --amt [AMOUNT_IN_SATS] --hops [LIST_OF_PUBLIC_KEYS_OF_PEERS] --outgoing_chan_id [OUTGOING_CHAN_ID] | lncli sendtoroute --payment_hash=[PAYMENT_HASH] -```


- `AMOUNT_IN_SATS`: for example 2000000
- `LIST_OF_PUBLIC_KEYS_OF_PEERS`: comma separated list of public keys of peers in order of the position in the ring
- `OUTGOING_CHAN_ID`: the ID of the outgoing channel you start with
- `PAYMENT_HASH`: the payment hash of your invoice

## Operations within the Ring

### Fee policy
Fee policies are a tool to directly influence routing behaviour in the network and can be core component for enacting specific strategies.

Fees are an open topic currently but policies will most likely be tailored to each ring depending on participant coun, channel size and common goals. . In most cases fees will be set the same between all the nodes within the ring.

The following strategies have been drafted out so far:

### Act as one big node

Any payment routed within the ring is free of charge. This turns the ring effectively into one big decentralised Lightning Node. Participants still earn routing fees through outgoing/incoming channels.

| Base fee     | Fee rate|
| :------------- | :----------: |
| 0 mSat | 0 mili mSat |

| Advantages     | Disadvantages|
| :------------- | :----------: |
| smaller players can band together to mimick big nodes | in comparison to a big node is that payments can require more hops which increases payment time and increase risk of payment failure   |



### Support micro payments
We set the base fee to 0 so smaller payments pay relatively less fees (i.e. 10 sats payment is not paying 1 sat / 10% in fees). Instead the flexible fee rate is increased to discourage big payments that can quickly throw channels out of balance.

| Base fee     | Fee rate|
| :------------- | :----------: |
| 0 mSat | >50 mili mSat |

| Advantages     | Disadvantages|
| :------------- | :----------: |
| channels can stay balanced longer | bigger payments can become costly |
| cheap micropayments | |

### Support big payments
If participants can and want to handle bigger payments, they can set the flexible fee rate lower (and possibly increase the base fees). This enabled larger payments through the ring without adding prohibitive costs.

| Base fee     | Fee rate|
| :------------- | :----------: |
| >0 mSat | <50 mili mSat |

| Advantages     | Disadvantages|
| :------------- | :----------: |
| bigger payments are cheaper | possibly discouraging micro payments |

## Migrating a node

If a participant finds the need to migrate the node to another instance. There are 2 ways to do so:

- copy the data folder and keep all channels
- starting from scratch

For LND, you can find the documentation here: https://github.com/lightningnetwork/lnd/blob/master/docs/safety.md#migrating-a-node-to-a-new-device

## Switching from clearnet to Tor

If the members of the ring opt for Tor only but a participants finds themself still on clearnet, the following guide can be followed for LND:
https://github.com/lightningnetwork/lnd/blob/master/docs/safety.md#migrating-a-node-from-clearnet-to-tor

### LN Maintenance

#### Keep backups

Static channel backups can save your funds in case your node crashes and does not recover. Backup regurarily, ideally after each opening and closing of a channel.

##### RTL
Ride the lightning let's you manually create and restore backups. Just head to `Lightning > Backup` and find the backup/download button. Store the backups in a redundant location

##### Automatic backups
There are a few scripts you can install to watch the data folder of LND and copy channel backups to another location.

**Your can use this script as a starting point to copy the backup to a different folder. Ideally to an external hard drive that is not used by LND**
https://gist.github.com/alexbosworth/2c5e185aedbdac45a03655b709e255a3

**Find an adaption of the script above which sends emails after each channel opening/closing**
https://github.com/Czino/lnd-channel-backup-2-mail


### Preventing data corruption
Sudden blackouts can be devastating to your data. LN nodes are constantly chatting with other nodes, this means they often update their database. If a node is interrupted in the middle of writing data, it might become corrupted. Should this happen, the safest way to recover your funds might only be to close all channels from your channel backup.
In order to avoid this situation, one should consider plugging a UPS (Uninterruptible Power Supply) inbetween.

Here's a good discussion about it: https://github.com/rootzoll/raspiblitz/issues/263#issue-397855893

If you already have or you are going to purchase a UPS without shutdown signal, you can use the following script as a workaround: https://github.com/Czino/graceful-shutdown
