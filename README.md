# Ring of Fire

Lightning is based on cooperation, but it is common that channel openings happen one-sided, (i. e. you are not in direct contact to your channel partner).

To strengthen cooperation and form strong partnerships, we have devised the concept of a ring of nodes, we dub *The Ring of Fire*. It serves as a template to bring node runners together and lift each other up.

## Benefits of participating in a Ring

Working on a common goal forms communities. If complications arise, participants can quickly reach out and receive help.

The connectivity of each node increases significantly as every other participants adds their channels to the outside.

We also currently investigate how and which data we share within the ring. This insight could give ring operators an edge over the rest of the users within the lightning network.

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

Note: to avoid channels being thrown out of balance before the initial balancing acts, ring members can decide to set **defensive fee policies** that make big payments prohibitevely expensive. A decent defensive policy that still allows for route building could be: `100 base / 5,000 ppm`.


#### Using LND

##### lncli
We can use the command line interface to open a channel to the next node.

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

Once the ring is closed (i.e. all channels are opened), one or more participants can create an invoice of half (or a fraction) of the channel capacity. The participant(s) will pay the invoice to themselves by building a route across the ring.

As hinted, this can be done by a single participant or more. If one participant takes over the task, they will send half of the channel capacity. If participants share the task then they send over `½ × channel capacity ÷ no. of participants`. The advantage of sharing the task is to offload the routing fees the members have to pay among each other, reducing the need for trust to be paid back.


Routes can be manually built by using the following tool

#### lncli

##### Build the route
First test if the route can be built. If a node is offline of the parameters are not correct, you will get informed. If successful, you will be presented with the information about the payment route, including the fees you will pay.

```lncli buildroute --amt [AMOUNT_IN_SATS] --hops [LIST_OF_PUBLIC_KEYS_OF_PEERS] --outgoing_chan_id [OUTGOING_CHAN_ID]```

- `AMOUNT_IN_SATS`: for example 2000000
- `LIST_OF_PUBLIC_KEYS_OF_PEERS`: comma separated list of public keys of peers in order of the position in the ring
- `OUTGOING_CHAN_ID`: the ID of the outgoing channel you start with

#### Troubleshooting
If the route cannot be built an error would be displayed f.e.:

`[lncli] rpc error: code = Unknown desc = no matching outgoing channel available for node 3 (0219ecc0ab49be9b91c3c302c99a32ddea784f352f329451e2ce7a7dcd461a684a)`

- You can check the channel between the node for which the NODE_PUBLIC_KEY is displayed and the next one in line with
  ```bash
  $ lncli getchaninfo CHAN_ID
  ```
- You can double-check **_the fee policies_** for the channels along the route. If the combined defensive fees that are originally set are too high then the route can also run out of funds when `buildroute` is run.


##### Pay yourself
Create an invoice of half of the channel capacity

Find the payment hash (not to be confused with the invoice)

*Find out the payment hash in lncli*

Use the command `lncli listinvoices` and read `r_hash` value and `payment_addr` of the corresponding invoice.

*Alternative: create invoice in lncli*
Use the command   `lncli addinvoice --amt [AMOUNT_IN_SATS] --expiry [TIME_IN_SECONDS]`

- `AMOUNT_IN_SATS`: for example 2000000
- `TIME_IN_SECONDS`: for example 3600 = 1 hour (is default value)

You can then use the payment hash and address to plug into the buildroute command

```lncli buildroute --amt [AMOUNT_IN_SATS] --hops [LIST_OF_PUBLIC_KEYS_OF_PEERS] --outgoing_chan_id [OUTGOING_CHAN_ID] | jq -r '(.route.hops[-1] | .mpp_record) |= {payment_addr:"[PAYMENT_ADDRESS]", total_amt_msat: "[AMOUNT_IN_MILLI_SATS]"}' | lncli sendtoroute --payment_hash=[PAYMENT_HASH] -```


- `AMOUNT_IN_SATS`: for example 2000000
- `LIST_OF_PUBLIC_KEYS_OF_PEERS`: comma separated list of public keys of peers in order of the position in the ring
- `OUTGOING_CHAN_ID`: the ID of the outgoing channel you start with
- `PAYMENT_ADDRESS`: the address of your invoice
- `PAYMENT_HASH`: the payment hash of your invoice

Fake example:
`lncli buildroute --amt 12345 --hops 02111eae1137fdeee8ca50633e21b46900ac6af458ef3eaaeadc4efafcd582d613,03672a5118121b4697b0bcf191ea65cc5a925e42b07449ef915a3ea99323a46adb,039dc42f731b519d18278022d321b469e91fbaf586f68ac6c4eeeaa1ccbbf7c739,03e121b469fc86eef047eeeb6102bd06f20bc2d035484bedaf07436b9543b1989 | jq -r '(.route.hops[-1] | .mpp_record) |= {payment_addr:"b717efdd6966f016421b46910ce3b50442a521b4692942fb0fc25022d267bdb", total_amt_msat: "12345000"}' | lncli sendtoroute --payment_hash=69f1521b469a5dd19b17a2932ec21b4699802d7d12c41b78c972dce9ac2f8dd -` 

#### Convenient Methods

There are scripts that abstract all of the above away. You can use the following (just make sure you understand the code and verify it does what it is intended to do)
- [Igniter by RooSoft](https://github.com/RooSoft/igniter)
- [Ring of Fire Terminal by Czino](https://github.com/Czino/ring-of-fire-sharing/tree/gui)

## Operations within the Ring

### Fee policy
Fee policies are a tool to directly influence routing behaviour in the network and can be core component for enacting specific strategies.

Fees are an open topic currently but policies will most likely be tailored to each ring depending on participant coun, channel size and common goals. . In most cases fees will be set the same between all the nodes within the ring.

The following strategies have been drafted out so far:

### Act as one big node

Any payment routed within the ring is free of charge. This turns the ring effectively into one big decentralised Lightning Node. Participants still earn routing fees through outgoing channels.

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


## Watchtowers

Lightning Nodes can offer to watch over other nodes' channels. They act as a "second line of defense" for the case your node is offline and another node intends to cheat on you by closing your channel with a wrong balance. A tower that watches over you can then broadcast a "justice transaction" that punished the attacker by sweeping all the funds and returning them to your node.

Because ring participants cooperate closely together, they can set up watch towers for each other.

Follow [the docs](https://github.com/lightningnetwork/lnd/blob/master/docs/watchtower.md) for further information.


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
In order to avoid this situation, one should consider plugging a UPS (Uninterruptible Power Supply) in between.

Here's a good discussion about it: https://github.com/rootzoll/raspiblitz/issues/263#issue-397855893

If you already have or you are going to purchase a UPS without shutdown signal, you can use the following script as a workaround: https://github.com/Czino/graceful-shutdown
