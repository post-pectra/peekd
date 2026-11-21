# Glossary

Terms as they are used in this repository's code, database, and documents.

## Ethereum and P2P terms

| Term | Meaning |
| --- | --- |
| Beacon chain | The Ethereum consensus layer chain whose P2P network peekd joins |
| CL node | Consensus layer node. peekd connects to the network without one |
| Slot | The beacon chain time unit. Its length comes from `SecondsPerSlot` in the chain configuration |
| Epoch | `SlotsPerEpoch` consecutive slots |
| Fork version | The 4-byte version of a consensus fork (Genesis, Altair, Bellatrix, Capella, Deneb, Electra). Handlers use it to choose the SSZ type of a message |
| Fork digest | The 4-byte value derived from the fork version and the genesis validators root. It is the segment after `eth2` in a topic string |
| DiscV5 | The UDP node discovery protocol used to find peers |
| ENR | Ethereum Node Record. The local record carries the IP, ports, subnet bitfields, and fork ID |
| Bootstrap node | A node from the network configuration that discovery contacts first |
| GossipSub | The libp2p publish/subscribe protocol that carries beacon chain messages |
| Topic | A GossipSub channel, named `/eth2/<fork digest>/<name>/ssz_snappy` |
| Global topic | A topic with a single channel, such as `beacon_block` |
| Subnet | One of several numbered channels of the same topic kind, such as `beacon_attestation_10` |
| Mesh degree (D, Dlo, Dhi) | Target, lower bound, and upper bound of the number of mesh peers per topic |
| Peer score | The GossipSub score of a peer, computed from the per-topic parameters in `gossip/peer_score.go` |
| Req/resp | The request/response protocols between beacon nodes (`ping`, `goodbye`, `status`, `metadata`) |
| SSZ-Snappy | The encoding of gossip and req/resp payloads: SSZ serialization with Snappy compression |

## peekd terms

| Term | Meaning |
| --- | --- |
| Watcher | The command and component that runs one peekd instance |
| Network | `mainnet` or `hoodi`. Other values are rejected at startup |
| Target peers | The peer count the dialer, the connection manager, and pruning aim for. The default is 80 |
| Dialer | The service that connects to nodes found by discovery |
| Prune | Closing connections to the lowest-scored peers above the target and to peers scored below -100 |
| Slot message | A message whose type carries a slot: block, blob sidecar, aggregate and proof, attestation, sync committee message, sync contribution and proof |
| General message | A message without a slot: proposer slashing, attester slashing, voluntary exit, BLS to execution change |
| Message ID | The GossipSub message ID computed by the Prysm `MsgID` function, stored hex-encoded |
| Topic group | The message kind without the subnet number, for example `beacon_attestation`. Column `topic_group` |
| Latency | Arrival time of the first copy minus the start time of the message's slot, in milliseconds. Column `latency_ms` |
| Seen count | How many times a message with the same ID was received. Column `seen_count`. A record with no counter entry is stored as 1 |
| Duplication | Seen count minus 1. Used in the hourly rollup |
| Completed slot | A slot the cache reports as ready to be written to the database |
| Node region, node alias | Labels that identify the instance that wrote a row. Columns `node_region` and `node_alias` |
| Hourly rollup | The table `slot_message_hourly_rollup`, filled once per hour by the materialized view |

## Easily confused

| Terms | Difference |
| --- | --- |
| `topic` column and topic | The column holds only the name segment of the topic string, for example `beacon_attestation_10`. The full topic also has the fork digest and the encoding suffix |
| `fork_version` column and fork version | The column holds the fork digest segment of the topic string, not the fork version |
| `topic` and `topic_group` | `topic` keeps the subnet number. `topic_group` does not |
| Node ID and peer ID | The node ID identifies the instance in DiscV5. The peer ID identifies it in libp2p. Both come from the same key |
| Target peers and mesh degree | Target peers limits connections for the whole host. Mesh degree limits mesh peers per topic |

## Message kinds

The processor puts each decoded message into one of two kinds. A message type that is in neither is rejected as unsupported.

| Kind | Stored in | When written |
| --- | --- | --- |
| Slot message | `slot_message_stats` | In a batch when its slot is flushed |
| General message | `general_message_histories` | On arrival |
