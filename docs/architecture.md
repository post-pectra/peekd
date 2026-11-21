# Architecture

The overview diagram and the key design decisions are in the [README](../README.md#architecture). This document adds what the diagram does not show. Terms are defined in the [glossary](glossary.md).

## Purpose and scope

peekd joins the Ethereum beacon chain P2P network as a libp2p peer, receives gossip messages, and records when each message arrived, how large it was, and how many times it was received. It needs no consensus layer (CL) node.

In scope:

- Peer discovery, connection, and GossipSub participation on `mainnet` and `hoodi`
- Per-message metadata written to ClickHouse
- Hourly aggregation inside ClickHouse

Out of scope:

- Validating messages. No topic validators are registered, so signatures and consensus rules are not checked.
- Publishing messages. The code never publishes to a topic.
- Storing message payloads. Only metadata is stored.
- Serving chain data to peers. The req/resp handlers answer only `ping`, `goodbye`, `status`, and `metadata`.
- Querying or visualizing the collected data.

## Components

One process runs the `watcher` command, which is also the default command. The watcher builds the components below and runs them as services under a single supervisor.

| Component | Responsibility |
| --- | --- |
| `watcher` | Reads options, selects the network, creates the node key, wires the other components together |
| `eth` | Holds the selected network for the whole process and derives genesis, fork version, subnet counts, and slot and epoch durations from the Prysm configuration |
| `host` | The libp2p host: TCP transport, Noise security, mplex and yamux multiplexers, IPv4 only. Reports inbound, outbound, and total peer counts |
| `peering` | Discovery finds nodes with DiscV5 and hands them to the dialer. The dialer connects to them through the host |
| `gossip` | Joins and subscribes to every topic, applies peer scoring, prunes peers once per slot, and passes each received message to the processor |
| `reqresp` | Registers stream handlers so that peers can complete the req/resp exchanges they expect from a beacon node |
| `processor` | Decodes a message, computes its metadata, caches slot messages by slot, and writes to the repository |
| `repository` | The storage interface and its ClickHouse implementation |

## Data flow

1. Discovery starts from the bootstrap nodes in the Prysm network configuration and iterates random nodes. Nodes with a private IP are skipped. Each remaining node is sent to the dialer over an unbuffered channel.
2. The dialer connects with a 2 second timeout. It pauses while the total peer count has reached the target and outbound peers are at least 20% of the target.
3. GossipSub delivers each message to the subscription service of its topic. The handler picks the SSZ type from the fork version and calls the processor.
4. The processor decodes the message and classifies it.
   - A slot message becomes one record in the in-memory cache of its slot. Latency is the arrival time minus the slot start time.
   - A general message is inserted into `general_message_histories` right away, in its own goroutine, with a 5 second timeout.
5. Separately, GossipSub calls the message ID function for each received copy of a message. That call increments the seen count for the ID.
6. A ticker fires once per slot duration. The processor drains the slots that the cache reports as completed, attaches the seen count to each record, and writes each slot as one batch insert into `slot_message_stats` with a 10 second timeout. It flushes once more on shutdown.
7. In ClickHouse, a refreshable materialized view runs every hour and appends aggregates of `slot_message_stats` to `slot_message_hourly_rollup`.

## External systems

| System | How peekd uses it |
| --- | --- |
| Ethereum beacon chain P2P network | DiscV5 over UDP and libp2p over TCP, both on port `9090` by default. The local ENR advertises all attestation and sync committee subnets |
| ClickHouse | Native protocol with LZ4 compression. With `PEEKD_DB_SECURE=true` the connection uses TLS without verifying the server certificate |
| Prysm libraries | Source of network configuration, topic formats, message types, the message ID function, and the SSZ-Snappy encoder |
| Docker Hub | The workflow pushes the image `a41ventures/peekd`. `docker-compose.yaml` pulls its `latest` tag on every start |
| SonarCloud | Scan on every push to `dev` and every `v*` tag |

## Data ownership

peekd owns three ClickHouse tables and one materialized view, all defined in [migrations/](../migrations).

| Object | Written by | Content |
| --- | --- | --- |
| `slot_message_stats` | processor, one batch per slot | One row per slot message |
| `general_message_histories` | processor, one insert per message | One row per general message |
| `slot_message_hourly_rollup` | materialized view `slot_message_hourly_mv` | Hourly latency and duplication aggregates. Rows expire after 30 days |

The application does not read from ClickHouse and has no migration runner. `docker-compose.yaml` mounts `migrations/` into the ClickHouse container's `/docker-entrypoint-initdb.d`.

Rows from several instances are told apart by `node_region` and `node_alias`.

## Operational boundaries

- **Fork handling is fixed at startup.** The topic list, its fork digest, and the fork version used to pick SSZ types are computed once when the process starts. Nothing re-subscribes at a fork boundary. The handlers know fork versions from genesis through Electra.
- **The slot cache is in memory only.** A slot is removed from the cache before the insert runs. If the insert fails, the error is logged and the records of that slot are lost.
- **The node key defines the peer identity.** Without `PEEKD_ECDSA_PRIVATE_KEY_HEX`, a new key is generated on each start, so the peer ID and node ID change on each restart.
- **`PEEKD_ESTIMATE_ACTIVE_VALIDATORS` is required.** GossipSub creation fails when it is 0. The value feeds the peer scoring parameters.
- **Mesh parameters differ from the mainnet spec.** The mesh degree is D=6, Dlo=5, Dhi=9. This changes first-arrival latency and seen count compared with a spec-default node.
- **Peer limits.** The connection manager starts trimming when connections exceed 1.25 times the target peer count and trims down to the target. Once per slot, the gossip component also closes the lowest-scored peers above the target and every peer whose score is below -100.
- **The hourly rollup skips incomplete hours.** An hour is aggregated for a node alias only when that alias has rows for 300 distinct slot start times in the hour. Rows with a latency of 768000 ms or more are excluded.
- **Pushing to `dev` publishes the image.** The same workflow run tags the image `latest`, which the compose file pulls.

## Related documents

- [Glossary](glossary.md)
- [ADR](adr/README.md)
